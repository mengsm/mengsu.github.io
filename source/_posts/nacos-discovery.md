---
title: Nacos注册中心
date: 2021-07-27 10:18:10
tags: Nacos
---

### **服务发现和服务健康监测**

Nacos 支持基于 DNS 和基于 RPC 的服务发现。服务提供者使用 [原生SDK](https://nacos.io/zh-cn/docs/sdk.html)、[OpenAPI](https://nacos.io/zh-cn/docs/open-api.html)、或一个[独立的Agent TODO](https://nacos.io/zh-cn/docs/other-language.html)注册 Service 后，服务消费者可以使用[DNS TODO](https://nacos.io/zh-cn/docs/xx) 或[HTTP&API](https://nacos.io/zh-cn/docs/open-api.html)查找和发现服务。

Nacos 提供对服务的实时的健康检查，阻止向不健康的主机或服务实例发送请求。Nacos 支持传输层 (PING 或 TCP)和应用层 (如 HTTP、MySQL、用户自定义）的健康检查。 对于复杂的云环境和网络拓扑环境中（如 VPC、边缘网络等）服务的健康检查，Nacos 提供了 agent 上报模式和服务端主动检测2种健康检查模式。Nacos 还提供了统一的健康检查仪表盘，帮助您根据健康状态管理服务的可用性及流量。

### 注册中心原理

在使用注册中心时，一共有三种角色：服务提供者（Service Provider）、服务消费者（Service Consumer）、注册中心（Registry）。

![点击](/images/nacos/nacos-dis.png)

1. Provider：

- 启动时，向 Registry **注册**自己为一个服务（Service）的实例（Instance）。
- 同时，定期向 Registry 发送**心跳**，告诉自己还存活。
- 关闭时，向 Registry **取消注册**。

2. Consumer：

- 启动时，向 Registry **订阅**使用到的服务，并缓存服务的实例列表在内存中。
- 后续，Consumer 向对应服务的 Provider 发起**调用**时，从内存中的该服务的实例列表选择一个，进行远程调用。
- 关闭时，向 Registry **取消订阅**。

3. Registry：

- Provider 超过一定时间未**心跳**时，从服务的实例列表移除。
- 服务的实例列表发生变化（新增或者移除）时，通知订阅该服务的 Consumer，从而让 Consumer 能够刷新本地缓存。

当然，不同的注册中心可能在实现原理上会略有差异。例如说，[Eureka](https://github.com/Netflix/eureka/) 注册中心，并不提供通知功能，而是 Eureka Client 自己定期轮询，实现本地缓存的更新。

另外，Provider 和 Consumer 是角色上的定义，一个服务**同时**即可以是 Provider 也可以作为 Consumer。例如说，优惠劵服务可以给订单服务提供接口，同时又调用用户服务提供的接口。

### 快速开始

#### 引入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>${spring.cloud.alibaba.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

#### 生产者

配置文件

```yaml
spring:
  application:
    name: demo-provider # Spring 应用名
  cloud:
    nacos:
      # Nacos 作为注册中心的配置项，对应 NacosDiscoveryProperties 配置类
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
        service: ${spring.application.name} # 注册到 Nacos 的服务名。默认值为 ${spring.application.name}。

server:
  port: 18080 # 服务器端口。默认为 8080
```

重点看 `spring.cloud.nacos.discovery` 配置项，它是 Nacos Discovery 配置项的前缀，对应 [NacosDiscoveryProperties](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-discovery/src/main/java/com/alibaba/cloud/nacos/NacosDiscoveryProperties.java) 配置项。



创建 DemoProviderApplication 类，创建应用启动类，并提供 HTTP 接口。代码如下：

```java
@SpringBootApplication
@EnableDiscoveryClient
public class DemoProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoProviderApplication.class, args);
    }

    @RestController
    static class TestController {

        @GetMapping("/echo")
        public String echo(String name) {
            return "provider:" + name;
        }

    }

}
```

1. `@SpringBootApplication` 注解，被添加在类上，声明这是一个 Spring Boot 应用。Spring Cloud 是构建在 Spring Boot 之上的，所以需要添加。

2. [`@EnableDiscoveryClient`](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/discovery/EnableDiscoveryClient.java) 注解，开启 Spring Cloud 的注册发现功能。不过从 Spring Cloud Edgware 版本开始，实际上已经不需要添加 `@EnableDiscoveryClient` 注解，只需要引入 Spring Cloud 注册发现组件，就会自动开启注册发现的功能。例如说，我们这里已经引入了 `spring-cloud-starter-alibaba-nacos-discovery` 依赖，就不用再添加 `@EnableDiscoveryClient` 注解了。

#### 消费者

配置文件

```yaml
spring:
  application:
    name: demo-provider # Spring 应用名
  cloud:
    nacos:
      # Nacos 作为注册中心的配置项，对应 NacosDiscoveryProperties 配置类
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
        service: ${spring.application.name} # 注册到 Nacos 的服务名。默认值为 ${spring.application.name}。

server:
  port: 28080 # 服务器端口。默认为 8080
```



创建 DemoConsumerApplication类，创建应用启动类，并提供一个调用服务提供者的 HTTP 接口。代码如下：

```java
@SpringBootApplication
// @EnableDiscoveryClient
public class DemoConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoConsumerApplication.class, args);
    }

    @Configuration
    public class RestTemplateConfiguration {

        @Bean
        public RestTemplate restTemplate() {
            return new RestTemplate();
        }

    }

    @RestController
    static class TestController {

        @Autowired
        private DiscoveryClient discoveryClient;
        @Autowired
        private RestTemplate restTemplate;
        @Autowired
        private LoadBalancerClient loadBalancerClient;

        @GetMapping("/hello")
        public String hello(String name) {
            // <1> 获得服务 `demo-provider` 的一个实例
            ServiceInstance instance;
            if (true) {
                // 获取服务 `demo-provider` 对应的实例列表
                List<ServiceInstance> instances = discoveryClient.getInstances("demo-provider");
                // 选择第一个
                instance = instances.size() > 0 ? instances.get(0) : null;
            } else {
                instance = loadBalancerClient.choose("demo-provider");
            }
            // <2> 发起调用
            if (instance == null) {
                throw new IllegalStateException("获取不到实例");
            }
            String targetUrl = instance.getUri() + "/echo?name=" + name;
            String response = restTemplate.getForObject(targetUrl, String.class);
            // 返回结果
            return "consumer:" + response;
        }

    }

}
```

1. `@EnableDiscoveryClient` 注解，因为已经无需添加，所以我们进行了注释，原因在上面已经解释过。

2. RestTemplateConfiguration 配置类，创建 [RestTemplate](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/web/client/RestTemplate.java) Bean。RestTemplate 是 Spring 提供的 HTTP 调用模板工具类，可以方便我们稍后调用服务提供者的 HTTP API。

3. TestController 提供了 `/hello` 接口，用于调用服务提供者的 `/demo` 接口。代码略微有几行，我们来稍微解释下哈。

`discoveryClient` 属性，DiscoveryClient 对象，服务发现客户端，上文我们已经介绍过。这里我们注入的不是 Nacos Discovery 提供的 NacosDiscoveryClient，保证通用性。未来如果我们不使用 Nacos 作为注册中心，而是使用 Eureka 或则 Zookeeper 时，则无需改动这里的代码。

`loadBalancerClient` 属性，[LoadBalancerClient](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/loadbalancer/LoadBalancerClient.java) 对象，负载均衡客户端。稍后我们会使用它，从 Nacos 获取的服务 `demo-provider` 的实例列表中，选择一个进行 HTTP 调用。

> 拓展小知识：在 Spring Cloud Common 项目中，定义了[LoadBalancerClient](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/loadbalancer/LoadBalancerClient.java) 接口，作为通用的负载均衡客户端，提供从指定服务中选择一个实例、对指定服务发起请求等 API 方法。而想要集成到 Spring Cloud 体系的负载均衡的组件，需要提供对应的 LoadBalancerClient 实现类。
>
> 例如说，Spring Cloud Netflix Ribbon 提供了 [RibbonLoadBalancerClient](https://github.com/spring-cloud/spring-cloud-netflix/blob/2.2.x/spring-cloud-netflix-ribbon/src/main/java/org/springframework/cloud/netflix/ribbon/RibbonLoadBalancerClient.java) 实现。
>
> 如此，所有需要使用到的地方，只需要获取到 DiscoveryClient 客户端，而无需关注具体实现，保证其通用性。😈 不过貌似 Spring Cloud 体系中，暂时只有 Ribbon 一个负载均衡组件。
>
> 当然，LoadBalancerClient 的服务的实例列表，是来自 DiscoveryClient 提供的。

`/hello` 接口，示例接口，对服务提供者发起一次 HTTP 调用。

- `<1>` 处，获得服务 `demo-provider` 的一个实例。这里我们提供了两种方式的代码，分别基于 DiscoveryClient 和 LoadBalancerClient。
- `<2>` 处，通过获取到的服务实例 ServiceInstance 对象，拼接请求的目标 URL，之后使用 RestTemplate 发起 HTTP 调用。



#### 测试

访问：http://127.0.0.1:28080/hello?name=mengsu ，返回结果为 `"consumer:provider:mengsu"`。说明，调用远程的**服务提供者**成功。



