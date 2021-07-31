---
title: springboot源码分析(02) - 项目结构详解
date: 2021-07-30 17:27:27
tags: SpringBoot
---

# 概述

本文主要分享 **Spring Boot 的项目结构**。
希望通过本文能让胖友对 Spring Boot 的整体项目有个简单的了解。

![](/images/springboot/springboot-root.png)



# spring-boot-project 项目

结构如下

![](/images/springboot/springboot-project.png)



## spring-boot 模块

`spring-boot` 模块，Spring Boot 的核心实现

- 在 `org.springframework.boot.SpringApplication` 类，提供了大量的静态方法，可以很容易运行一个独立的 Spring 应用程序。

  > 经常使用。

- 带有可选容器的嵌入式 Web 应用程序（Tomcat、Jetty、Undertow） 的支持。

  > 在 `org.springframework.boot.web` 包下实现。

- 边界的外部配置支持。

- … 省略其它。



## spring-boot-autoconfigure 模块

`spring-boot-autoconfigure` 可以根据类路径的内容，自动配置大部分常用应用程序。通过使用 `org.springframework.boot.autoconfigure.@EnableAutoConfiguration` 注解，会触发 Spring 上下文的自动配置。

> 这里的大部分，指的是常用的框架。例如说，Spring MVC、Quartz 等等。也就是说，如果 `spring-boot-actuator-autoconfigure` 模块，暂未提供的框架，需要我们自己去实现对应框架的自动装配。

这个模块的代码，必须要看，没得商量。

所以到此处为止，我们已经看到对我们来研究 Spring Boot 最最最重要的两个模块：`spring-boot` 和 `spring-boot-autoconfigure` 。



## spring-boot-actuator 模块

`spring-boot-actuator` 模块。正如其模块的英文 actuator ，它完全是一个用于暴露应用自身信息的模块：

- 提供了一个监控和管理生产环境的模块，可以使用 http、jmx、ssh、telnet 等管理和监控应用。
- 审计（Auditing）、 健康（health）、数据采集（metrics gathering）会自动加入到应用里面。

> 一般情况下，我们可以不看这块代码的代码。



## spring-boot-actuator-autoconfigure 模块

`spring-boot-actuator-autoconfigure` 模块，大概 1W7 行代码左右。它提供了 `spring-boot-actuator` 的自动配置功能。

> 一般情况下，我们可以不看这块代码的代码。



## spring-boot-starters 模块

`spring-boot-starters` 模块，它不存在任何的代码，而是提供我们常用框架的 Starter 模块。例如：



- `spring-boot-starter-web` 模块，提供了对 Spring MVC 的 Starter 模块。
- `spring-boot-starter-data-jpa` 模块，提供了对 Spring Data JPA 的 Starter 模块。



而每个 Starter 模块，里面只存在一个 `pom` 文件，这是为什么呢？简单来说，Spring Boot 可以根据项目中是否存在指定类，并且是否未生成对应的 Bean 对象，那么就自动创建 Bean 对象。因为有这样的机制，我们只需要使用 `pom` 文件，配置需要引入的框架，就可以实现该框架的使用所需要的类的自动装配。



> 当然，如果其中没有自己想要的starter， 可以自定义实现。



## spring-boot-cli 模块



`spring-boot-cli` 模块，大概 1W 行代码左右。它提供了 Spring 项目相关的命令行功能。它是 Spring Boot 的命令行界面。

- 它可以用来快速启动 Spring 。
- 它可以运行 Groovy 脚本，开发人员不需要编写很多样板代码，只需要关注业务逻辑。
- Spring Boot CLI 是创建基于Spring的应用程序的最快方法。



> 一般情况下，我们可以不看这块代码的代码。



## spring-boot-test 模块



`spring-boot-test` 模块，大概 1W 行代码左右。Spring Boot 提供测试方面的支持，例如说：

- SpringBootTestRandomPortEnvironmentPostProcessor 类，提供随机端口。
- `org.springframework.boot.test.mock.mockito` 包，提供 Mockito 的增强。



> 一般情况下，我们可以不看这块代码的代码。



## spring-boot-test-autoconfigure 模块



`spring-boot-test-autoconfigure` 模块，大概 1W 行代码不到。它提供了 `spring-boot-test` 的自动配置功能。

> 一般情况下，我们可以不看这块代码的代码。



## spring-boot-devtools 模块

`spring-boot-devtools` 模块，大概 8000 行代码左右。通过它，来使 Spring Boot 应用支持热部署，提高开发者的开发效率，无需手动重启 Spring Boot 应用。



> 一般情况下，我们可以不看这块代码的代码。



## spring-boot-tools 模块

`spring-boot-tools` 模块，大概 3W 行代码左右。它是 Spring Boot 提供的工具箱，所以在其内有多个子 Maven 项目。

注意哟，我们这里说的工具箱，并不是我们在 Java 里的工具类。困惑？我们来举个例子：`spring-boot-maven-plugin` 模块：提供 Maven 打包 Spring Boot 项目的插件。

关于 `spring-boot-tools` 模块的其它子模块，我们就暂时不多做介绍落。

> 一般情况下，我们可以不看这块代码的代码。



# spring-boot-samples 项目

`spring-boot-samples` 项目，2W 行代码左右。丧心病狂，提供了超级多的示例，简直良心无敌啊。

> 一般情况下，我们可以不看这块代码的代码。如果真的需要某个 Spring Boot 对某个框架的示例，大多数情况下，我们还是 Google 检索文章居多。



# spring-boot-samples-invoker 项目

`spring-boot-samples-invoker` 项目，无代码，不用看。当然，也并不重要。



# spring-boot-tests

`spring-boot-tests` 项目，3000 行代码，主要是 Spring Boot 的集成测试、部署测试。

> 一般情况下，我们可以不看这块代码的代码。





# 结尾



所以我们重点关注的应该是： spring-boot-project ： `spring-boot`  和 `spring-boot-autoconfigure` 和  `spring-boot-starters`
