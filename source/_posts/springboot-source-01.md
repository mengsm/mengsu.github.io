---
title: springboot源码分析(01) - 搭建调试环境
date: 2021-07-29 17:27:27
tags: SpringBoot
---

# 前言

为了更加深入理解`springboot`，近期开始阅读`springboot`的源码，话不多说，开干！



# 依赖工具

- Maven (springboot好多版本都是使用Gradle构建，搭建的胖友们需注意)
- Git
- Jdk8+
- IntelliJ IDEA



# 源码拉取

从官方仓库 https://github.com/spring-projects/spring-boot `Fork` 出属于自己的仓库。



- 为什么要 `Fork` ？既然开始阅读、调试源码，我们可能会写一些注释，有了自己的仓库，可以进行自由的提交。
- 本文使用的 springboot 版本为 `2.1.19.BUILD-SNAPSHOT` 。
- 使用 `IntelliJ IDEA` 从 `Fork` 出来的仓库拉取代码。因为 Spring 项目比较大，从仓库中拉取代码的时间会比较长。



拉取完成后，Maven 会开始自动 **Build** 项目。因为 Build 的过程中，会下载非常多的依赖，请耐心等待。



# 构建调试Demo

## 解决 pom 的报错

在根目录的 `pom.xml` 中，会看到 `${disable.checks}` 报错。它是用来配置，是否开启 Maven 代码检查的插件。因为，我们目的是为了调试代码，所以自然是去禁用它。仅仅需要在 `pom.xml` 配置如下：

```xml
	<properties>
		<revision>2.1.19.BUILD-SNAPSHOT</revision>
		<main.basedir>${basedir}</main.basedir>
		<disable.checks>true</disable.checks>    <!-- 我是被加的 -->
	</properties>
```



## 搭建 MVC 调试环境

在 `spring-boot-tests`  下新模块 `spring-boot-xxx-tests`  >  以后自己测试的模块都是放在里面, 在新建的模块下面再新建 `spring-boot-xxx-mvc-tests` 



spring-boot-xxx-tests pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<parent>
		<artifactId>spring-boot-tests</artifactId>
		<groupId>org.springframework.boot</groupId>
		<version>${revision}</version>
	</parent>
	<modelVersion>4.0.0</modelVersion>

	<artifactId>spring-boot-mengsu-tests</artifactId>
	<packaging>pom</packaging>
	<modules>
		<module>spring-boot-xxx-mvc-tests</module>
	</modules>

	<description>我的Spring Boot测试模块</description>

	<properties>
		<main.basedir>${basedir}/../..</main.basedir>
		<java.version>1.8</java.version>
	</properties>

</project>
```



spring-boot-xxx-mvc-tests pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<parent>
		<artifactId>spring-boot-mengsu-tests</artifactId>
		<groupId>org.springframework.boot</groupId>
		<version>${revision}</version>
	</parent>
	<modelVersion>4.0.0</modelVersion>

	<artifactId>spring-boot-mengsu-mvc-tests</artifactId>
	<name>我的Spring Boot测试模块 之 MVC部分</name>
	<description>${project.name}</description>

	<properties>
		<main.basedir>${basedir}/../../..</main.basedir>
	</properties>

	<dependencies>
		<!-- Compile -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```



新建Application 启动类

```java
package site.mengsu;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author: mengsu
 * @Date: 2021/7/30
 * @Description:
 */
@RestController
@SpringBootApplication
public class MyTestMVCApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyTestMVCApplication.class, args);
	}

	@GetMapping
	public String index () {
		return "success";
	}
}
```



## 测试运行

`MyTestMVCApplication` 启动，访问：http://localhost:8080/

返回：success，即为成功。
