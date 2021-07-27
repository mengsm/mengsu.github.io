---
title: Nacos配置中心
date: 2021-07-27 13:28:22
tags: Nacos
---

# [概念](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config#spring-cloud-alibaba-nacos-config)

Nacos 提供用于存储配置和其他元数据的 key/value 存储，为分布式系统中的外部化配置提供服务器端和客户端支持。使用 Spring Cloud Alibaba Nacos Config，您可以在 Nacos Server 集中管理你 Spring Cloud 应用的外部属性配置。

Spring Cloud Alibaba Nacos Config 是 Config Server 和 Client 的替代方案，客户端和服务器上的概念与 Spring Environment 和 PropertySource 有着一致的抽象，在特殊的 bootstrap 阶段，配置被加载到 Spring 环境中。当应用程序通过部署管道从开发到测试再到生产时，您可以管理这些环境之间的配置，并确保应用程序具有迁移时需要运行的所有内容。



# 快速开始

## 引入依赖

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
   <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

## 配置文件

创建 `bootstrap.yaml`配置文件，添加 Nacos Config 相关配置。配置如下：

```yaml
spring:
  application:
    name: demo-application

  cloud:
    nacos:
      # Nacos Config 配置项，对应 NacosConfigProperties 配置属性类
      config:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
        namespace: # 使用的 Nacos 的命名空间，默认为 null
        group: DEFAULT_GROUP # 使用的 Nacos 配置分组，默认为 DEFAULT_GROUP
        name: # 使用的 Nacos 配置集的 dataId，默认为 spring.application.name
        file-extension: yaml # 使用的 Nacos 配置集的 dataId 的文件拓展名，同时也是 Nacos 配置集的配置格式，默认为 properties

```

Nacos Config 配置项，以 `spring.cloud.nacos.config` 开头，对应 [NacosConfigProperties](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-config/src/main/java/com/alibaba/cloud/nacos/NacosConfigProperties.java) 配置属性类。

1. `server-addr` 配置项，设置 Nacos 服务器地址。

2. `namespace` 配置项，使用的 Nacos 的命名空间，默认为 `null`，表示使用 `public` 这个默认命名空间。

> FROM [《Nacos 文档 —— Nacos 概念》](https://nacos.io/zh-cn/docs/concepts.html)
>
> **命名空间**
> 用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。Namespace 的常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

3. `group` 配置项，使用的 Nacos 配置分组，默认为 `DEFAULT_GROUP`。

> FROM [《Nacos 文档 —— Nacos 概念》](https://nacos.io/zh-cn/docs/concepts.html)
>
> **配置分组**
> Nacos 中的一组配置集，是组织配置的维度之一。通过一个有意义的字符串（如 Buy 或 Trade ）对配置集进行分组，从而区分 Data ID 相同的配置集。当您在 Nacos 上创建一个配置时，如果未填写配置分组的名称，则配置分组的名称默认采用 `DEFAULT_GROUP` 。配置分组的常见场景：不同的应用或组件使用了相同的配置类型，如 `database_url` 配置和 `MQ_topic` 配置。

4. `name` 配置项，使用的 Nacos 配置集的 dataId，默认为 `spring.application.name`。

> FROM [《Nacos 文档 —— Nacos 概念》](https://nacos.io/zh-cn/docs/concepts.html)
>
> **配置集**
> 一组相关或者不相关的配置项的集合称为配置集。在系统中，一个配置文件通常就是一个配置集，包含了系统各个方面的配置。例如，一个配置集可能包含了数据源、线程池、日志级别等配置项。
>
> **配置集 ID**
> Nacos 中的某个配置集的 ID。配置集 ID 是组织划分配置的维度之一。Data ID 通常用于组织划分系统的配置集。一个系统或者应用可以包含多个配置集，每个配置集都可以被一个有意义的名称标识。Data ID 通常采用类 Java 包（如 `com.taobao.tc.refund.log.level`）的命名规则保证全局唯一性。此命名规则非强制。

因为这里我们未进行配置，所以使用 Nacos 配置集的 dataId 为 `demo-application`。这也是为什么我们将 `spring.application.name` 配置项添加到 `bootstrap.yaml` 配置文件中的原因。

5. `file-extension` 配置项，使用的 Nacos 配置集的 dataId 的**文件拓展名**，同时也是 Nacos 配置集的**配置格式**，默认为 `properties`。这里我们设置为 `yaml`，因为我们稍后使用的配置集的配置格式为 `YAML`。



### bootstrap.yaml

> 为什么要将 Nacos Config 的配置项添加到 `bootstrap.yaml` 配置文件，而不像我们之前的一样，添加到 `application.yaml` 配置文件中呢？
>
> 下面，我们来讲解下原因

在 Spring Cloud 应用中，会先创建一个 **Bootstrap** Context（**引导**上下文），比 Spring Boot 创建 **Application** Context（**应用**上下文）**更早初始化**。

Bootstrap Context 新增了一个 `bootstrap.yaml` **配置文件**，保证和 Application Context 的 `application.yaml` **配置文件**的**隔离**。

有了配置文件的隔离之后，Bootstrap Context 初始化的 **Bean** 从哪里来？Spring Cloud 新定义了专属于 Bootstrap Context 的自动化配置类的拓展点 **BootstrapConfiguration**，和 Spring Boot 为 Application Context 的自动化配置类的拓展点 **EnableAutoConfiguration**的**隔离**，保证两个 Context 创建各自的 Bean。

虽然说，Bootstrap Context 和 Application Context 做了这么多隔离，但是它们有一点是共享的，那就是 **Environment**。在 Spring 中，我们通过 Environment 获取属性配置，例如说 `spring.application.name` 对应的值是多少。

了解完这些之后，我们把它们串联在一起去思考一下，Bootstrap Context 的**目的**究竟是什么呢？通过 Bootstrap Context 的优先初始化，**将配置加载到 Environment 中**，提供给后面的 Application Context 使用。

举个贼重要的例子，稍后我们会在 `bootstrap.yaml` 添加 Spring Cloud Alibaba Nacos Config 相关的配置，这样 Bootstrap Context 在初始化时，通过 NacosConfigBootstrapConfiguration 创建 Nacos 相关的 Bean，然后实现从 Nacos 配置中心加载配置到 Environment 中。

如果我们把 Spring Cloud Alibaba Nacos Config 相关的配置添加在 `application.yaml` 中，那么可能无法保证 Nacos 相关的 Bean 被**最先**初始化，完成从 Nacos 获取配置，从而影响创建的 Bean。



## 创建 Nacos 配置集

打开 Nacos UI 界面的「配置列表」菜单，进入「配置管理」功能。如下图所示：

![](/images/nacos/nacos-config-save.png)



## 编写代码

创建 OrderProperties配置类，读取 `order` 配置项。代码如下：

```java
@Component
@ConfigurationProperties(prefix = "order")
// @NacosConfigurationProperties(prefix = "order", dataId = "${nacos.config.data-id}", type = ConfigType.YAML)
public class OrderProperties {

    /**
     * 订单支付超时时长，单位：秒。
     */
    private Integer payTimeoutSeconds;

    /**
     * 订单创建频率，单位：秒
     */
    private Integer createFrequencySeconds;

    // ... 省略 setter/getter 方法

}
```



- 在类上，添加 `@Component` 注解，保证该配置类可以作为一个 Bean 被扫描到。
- 在类上，添加 `@ConfigurationProperties` 注解，并设置 `prefix = "order"` 属性，这样它就可以读取**前缀**为 `order` 配置项，设置到配置类对应的属性上。





创建 DemoController类，提供测试 `@ConfigurationProperties` 和 `@Value` 注入配置的两个 HTTP 接口。代码如下：

```java
@RestController
@RequestMapping("/demo")
public class DemoController {

    @Autowired
    private OrderProperties orderProperties;

    /**
     * 测试 @ConfigurationProperties 注解的配置属性类
     */
    @GetMapping("/test01")
    public OrderProperties test01() {
        return orderProperties;
    }

    @Value(value = "${order.pay-timeout-seconds}") // @NacosValue(value = "${order.pay-timeout-seconds}")
    private Integer payTimeoutSeconds;
    @Value(value = "${order.create-frequency-seconds}") // @NacosValue(value = "${order.create-frequency-seconds}")
    private Integer createFrequencySeconds;

    /**
     * 测试 @Value 注解的属性
     */
    @GetMapping("/test02")
    public Map<String, Object> test02() {
        return new JSONObject().fluentPut("payTimeoutSeconds", payTimeoutSeconds)
                .fluentPut("createFrequencySeconds", createFrequencySeconds);
    }

}
```

## 简单测试

访问 http://127.0.0.1:8080/demo/test01 接口

访问 http://127.0.0.1:8080/demo/test02 接口

结果：

```json
{
    "payTimeoutSeconds": 60,
    "createFrequencySeconds": 120
}
```



# 自动刷新配置

- 使用 `@ConfigurationProperties` 注解，使用 `@Value` 注解的**不会**
- 使用 `@RefreshScope` 注解，



## EnvironmentChangeEvent

通过 `@ConfigurationProperties` 或者 `@Value` + `@RefreshScope` 注解，已经能够满足我们绝大多数场景下的自动刷新配置的功能。但是，在一些场景下，我们仍然需要**实现对配置的监听，执行自定义的逻辑**。

例如说，当数据库连接的配置发生变更时，我们需要通过监听该配置的变更，重新初始化应用中的数据库连接，从而访问到新的数据库地址。

```java
@Component
public class DemoEnvironmentChangeListener implements ApplicationListener<EnvironmentChangeEvent> {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private ConfigurableEnvironment environment;

    @Override
    public void onApplicationEvent(EnvironmentChangeEvent event) {
        for (String key : event.getKeys()) {
            logger.info("[onApplicationEvent][key({}) 最新 value 为 {}]", key, environment.getProperty(key));
        }
    }

}
```



# 官方文档

- [《Nacos 官方文档》](https://nacos.io/zh-cn/docs/what-is-nacos.html)
- [《Spring Cloud Alibaba 官方**文档** —— Nacos Config》](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config)
- [《Spring Cloud Alibaba 官方**示例** —— Nacos Config》](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-examples/nacos-example/nacos-config-example/readme-zh.md)

