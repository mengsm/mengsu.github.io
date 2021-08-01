---
title: springboot源码分析(03) - SpringApplication构造方法
date: 2021-07-31 16:36:50
tags: SpringBoot
---

# 概述

上节自己搭建的 `web` 例子

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @Author: mengsu
 * @Date: 2021/7/30
 * @Description:
 */
@SpringBootApplication // <1>
public class MyTestMVCApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyTestMVCApplication.class, args);  // <2>
	}

}
```

- `<1>` 处，使用 `@SpringBootApplication` 注解，标明是 Spring Boot 应用。通过它，可以开启自动配置的功能。
- `<2>` 处，调用 `SpringApplication#run(Class<?>... primarySources)` 方法，启动 Spring Boot 应用。

上述的代码，是我们使用 Spring Boot 时，最最最常用的代码。而本文，我们先来分析 Spring Boot 应用的**启动过程**。



### SpringApplication类注释

```java
/**
 * Class that can be used to bootstrap and launch a Spring application from a Java main
 * method. By default class will perform the following steps to bootstrap your
 * application:
 *
 * <ul>
 * <li>Create an appropriate {@link ApplicationContext} instance (depending on your
 * classpath)</li>
 * <li>Register a {@link CommandLinePropertySource} to expose command line arguments as
 * Spring properties</li>
 * <li>Refresh the application context, loading all singleton beans</li>
 * <li>Trigger any {@link CommandLineRunner} beans</li>
 * </ul>
 *
 * In most circumstances the static {@link #run(Class, String[])} method can be called
 * directly from your {@literal main} method to bootstrap your application:
 *
 * <pre class="code">
 * &#064;Configuration
 * &#064;EnableAutoConfiguration
 * public class MyApplication  {
 *
 *   // ... Bean definitions
 *
 *   public static void main(String[] args) {
 *     SpringApplication.run(MyApplication.class, args);
 *   }
 * }
 * </pre>
 *
 * <p>
 * For more advanced configuration a {@link SpringApplication} instance can be created and
 * customized before being run:
 *
 * <pre class="code">
 * public static void main(String[] args) {
 *   SpringApplication application = new SpringApplication(MyApplication.class);
 *   // ... customize application settings here
 *   application.run(args)
 * }
 * </pre>
 *
 * {@link SpringApplication}s can read beans from a variety of different sources. It is
 * generally recommended that a single {@code @Configuration} class is used to bootstrap
 * your application, however, you may also set {@link #getSources() sources} from:
 * <ul>
 * <li>The fully qualified class name to be loaded by
 * {@link AnnotatedBeanDefinitionReader}</li>
 * <li>The location of an XML resource to be loaded by {@link XmlBeanDefinitionReader}, or
 * a groovy script to be loaded by {@link GroovyBeanDefinitionReader}</li>
 * <li>The name of a package to be scanned by {@link ClassPathBeanDefinitionScanner}</li>
 * </ul>
 *
 * Configuration properties are also bound to the {@link SpringApplication}. This makes it
 * possible to set {@link SpringApplication} properties dynamically, like additional
 * sources ("spring.main.sources" - a CSV list) the flag to indicate a web environment
 * ("spring.main.web-application-type=none") or the flag to switch off the banner
 * ("spring.main.banner-mode=off").
 *
 * @author Phillip Webb
 * @author Dave Syer
 * @author Andy Wilkinson
 * @author Christian Dupuis
 * @author Stephane Nicoll
 * @author Jeremy Rickard
 * @author Craig Burke
 * @author Michael Simons
 * @author Madhura Bhave
 * @author Brian Clozel
 * @author Ethan Rubinson
 * @since 1.0.0
 * @see #run(Class, String[])
 * @see #run(Class[], String[])
 * @see #SpringApplication(Class...)
 */
```

`SpringApplication` 用于从 `java main` 方法引导和启动 `Spring` 应用程序，默认情况下，将执行以下步骤来引导我们的应用程序：

1. 创建一个恰当的 `ApplicationContext` 实例（取决于类路径）
2. 注册 `CommandLinePropertySource` ，将命令行参数公开为 `Spring` 属性
3. 刷新应用程序上下文，加载所有单例 `bean`
4. 触发全部 `CommandLineRunner bean`



大多数情况下，像 `SpringApplication.run(MyTestMVCApplication, args)` ; 这样启动我们的应用，也可以在运行之前创建和自定义 `SpringApplication` 实例，具体可以参考注释中示例。`SpringApplication` 可以从各种不同的源读取bean。 通常建议使用单个 `@Configuration` 类来引导，但是我们也可以通过以下方式来设置资源：

1. 通过 `AnnotatedBeanDefinitionReader` 加载完全限定类名
2. 通过 `XmlBeanDefinitionReader` 加载 XML 资源位置，或者是通过 `GroovyBeanDefinitionReader` 加载 `groovy` 脚本位置
3. 通过 `ClassPathBeanDefinitionScanner` 扫描包名称

也就是说 `SpringApplication` 还是做了不少事的，具体实现后续会慢慢讲来，今天的主角只是 `SpringApplication` 构造方法。



# SpringApplication构造方法



```java
 	public static void main(String[] args) { // <1>
		SpringApplication.run(MyTestMVCApplication.class, args);   
	}
	
	public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) { // <2>
		return run(new Class<?>[] { primarySource }, args);				
	}
	
	public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) { // <3>
		return new SpringApplication(primarySources).run(args);   
	}
	
	public SpringApplication(Class<?>... primarySources) { // <4>
		this(null, primarySources);		
	}
	
	/**
	 * Create a new {@link SpringApplication} instance. The application context will load
	 * beans from the specified primary sources (see {@link SpringApplication class-level}
	 * documentation for details. The instance can be customized before calling
	 * {@link #run(String...)}.
	 * @param resourceLoader the resource loader to use
	 * @param primarySources the primary bean sources
	 * @see #run(Class, String[])
	 * @see #setSources(Set)
	 */
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) { // <5>
		this.resourceLoader = resourceLoader;   
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources)); 
		this.webApplicationType = WebApplicationType.deduceFromClasspath(); // <5.1>
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));  // <5.2>
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class)); // <5.3>
		this.mainApplicationClass = deduceMainApplicationClass();  // <5.4>
	}
```

按照层级跳转分为以上五个层级结构，可以发现咱们自定义的 `MyTestMVCApplication main` 方法都是调用的 `SpringApplication` 静态方法，直接到达的构造函数。

从 `SpringApplication` 构造函数的注释上来看，就是说创建一个 `MyTestMVCApplication `实例，应用上下文从特定的资源文件中加载`bean`。可以在调用`run`之前自定义实例。



## 5.1 WebApplicationType.deduceFromClasspath()

```java
	static WebApplicationType deduceFromClasspath() {
		if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
				&& !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
			return WebApplicationType.REACTIVE;
		}
		for (String className : SERVLET_INDICATOR_CLASSES) {
      // 判断给定的类是否能够加载，就是说类路径下是否存在给定的类
			if (!ClassUtils.isPresent(className, null)) {
				return WebApplicationType.NONE;
			}
		}
		return WebApplicationType.SERVLET;
	}
```

> 如果`org.springframework.web.reactive.DispatcherHandler`能够被加载且`org.springframework.web.servlet.DispatcherServlet`不能够被加载，那么断定`web`应用类型是`REACTIVE`；如果`javax.servlet.Servlet`和`org.springframework.web.context.ConfigurableWebApplicationContext`任意一个不能被加载，那么断定`web`应用类型是`NONE`；如果不能断定是`REACTIVE`和`NONE`，那么就是`SERVLET`类型；具体这三种类型代表什么含义，可以查看WebApplicationType中的说明。　　



## 5.2 setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));

> 从字面意思看就是获取spring工厂实例，至于从哪获取哪些工厂实例，我们往下看。

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) { // <1>
		return getSpringFactoriesInstances(type, new Class<?>[] {});
}

private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) { // <2>
		ClassLoader classLoader = getClassLoader();
		// Use names and ensure unique to protect against duplicates
		Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
}
```

### SpringFactoriesLoader.loadFactoryNames

> **SpringFactoriesLoader.loadFactoryNames** 可以先留意下，该处代码会在 `run` 的时候进行读取，可以看到现在的作用就是为了初始化实例，并且放在缓存当中。
>
> 1. 查找类路径下全部的META-INF/spring.factories的URL
>
> 2. 根据url加载全部的spring.factories中的属性，spring.factories内容如下
>
> 3. 将所有spring.factories中的值缓存到SpringFactoriesLoader的cache中：
>
>    private static final Map<ClassLoader, MultiValueMap<String, String>> cache = new ConcurrentReferenceHashMap<>()；方便下次调用。

```java
	public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
		String factoryClassName = factoryClass.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
	}
	
	private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
      // classLoader.getResources(FACTORIES_RESOURCE_LOCATION)获取类路径下全部的META-INF/spring.factories的URL
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
      // 遍历全部的URL，逐个读取META-INF/spring.factories中的属性
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					String factoryClassName = ((String) entry.getKey()).trim();
          // 属性全部放入MultiValueMap<String, String> result中，注意result的类型
					for (String factoryName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
						result.add(factoryClassName, factoryName.trim());
					}
				}
			}
      // 结果放入缓存，方便下次查找，run方法中会使用缓存
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```

### createSpringFactoriesInstances

> 根据上面获取的指定类型的工厂名称列表来实例化工厂bean，我们可以简单的认为通过反射来实例化，但是具体的实现也没那么简单，感兴趣的可以自己去跟。

### AnnotationAwareOrderComparator.sort(instances)

> 排序规则：@Order从小到大排序，没有order则按没排序之前的顺序。



## 5.3 getSpringFactoriesInstances(ApplicationListener.class))

> 类似 5.2 一样，  只是传入的类型不同 > `ApplicationListener`



# 总结

1. 构造自身实例
2. 推测web应用类型，并赋值到属性webApplicationType
3.  设置属性 

```java
List<ApplicationContextInitializer<?>> initializers
List<ApplicationListener<?>> listeners
```

中途读取了类路径下所有META-INF/spring.factories的属性，并缓存到了SpringFactoriesLoader的cache缓存中

4. 推断主类，并赋值到属性mainApplicationClass
