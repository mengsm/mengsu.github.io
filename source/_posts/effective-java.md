---
title: Effective Java 笔记
date: 2021-08-09 23:50:45
tags: Effective Java
---

# 一、用静态工厂方法代替构造器

### 优点

1. 静态工厂方法与构造器不同的第一大优势在于，他们有名称

> 比如 BigInteger.probablePrime 直接返回可能为素数

2. 静态工厂方法与构造器不同的第二大优势在于，不必在每次调用它们的时候都创建一个新对象

> 可以使用预先构建好的实例，或者将构建好的实例缓存起来， 进行重复利用，从而避免创建不必要的重复对象

3. 静态工厂方法与构造器不同的第三大优势在于，它们可以返回原返回类型的任何子类型的对象

> 例如 Java Collections Framework 的集合接口有 45 个工具实现，分别提供了不可修改的集合、 同步集合，等等

4. 静态工厂的第四大优势在于，所返回的对象的类可以随着每次调用而发生变化，这取决于静态工厂方法的参数值

> ..

5. 静态工厂的第五大优势在于，方法返回的对象所属的类，在编写包含该静态工厂方法的类时可以不存在

```java
/** 返回Connection,该类为DriverManager.class */
    @CallerSensitive
    public static Connection getConnection(String url,
        java.util.Properties info) throws SQLException {

        return (getConnection(url, info, Reflection.getCallerClass()));
    }
```

### 缺点

1. 静态工厂方法的主要缺点在子，类如果不含公有的或者受保护的构造器，就不能被子类化
2. 静态工厂方法的第二个缺点在于，程序员很难发现它们

> 对于提供了静态工厂方法而不是构造器的类来说，要想查明如何实例化一个类是非常困难的 。



## 总结

> 简而言之，静态工厂方法和公有构造器都有用处，我们需要理解他们各自的长处。静态工厂经常更加合适，因此切忌第一反应就是提供公有的构造器，而不先考虑静态工厂。





# 二、遇到多个构造器参数时要考虑使用构建器

> 静态工厂和构造器有个共同的局限性：它们都不能很好 扩展到大量的可选参数
>
> 
>
> 重叠构造器模式可行，但是当有许多参数的时候，客户端代码会很难缩写，并且仍然较难以阅读
>
> 
>
> 简而言之 如果类的构造器或者静态工厂中具有多个参数，设计这种类时， Builde模式就是一种不错的选择，
>
> 采用 `Buide` 模式



# 三、用私有构造器或者枚举类型强化 Singleton 属性









