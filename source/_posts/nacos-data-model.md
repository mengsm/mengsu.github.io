---
title: Nacos数据模型
date: 2021-07-27 11:24:21
tags: Nacos
---

## 数据模型

Nacos 数据模型 Key 由三元组唯一确认。如下图所示：

![点击](/images/nacos/nacos-data-model.jpeg)

- 作为注册中心时，Namespace + Group + Service
- 作为配置中心时，Namespace + Group + DataId

## Namespace 命名空间

用于进行租户粒度的配置隔离。默认为 public（公共命名空间）。

不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。Namespace 的常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

## Group 服务分组

不同的服务可以归类到同一分组。默认为 DEFAULT_GROUP（默认分组）。

## Service 服务

例如说，用户服务、订单服务、商品服务等等。

## 官方文档

- [《Nacos 官方文档 —— 概念》](https://nacos.io/zh-cn/docs/what-is-nacos.html)
- [《Nacos 官方文档 —— 架构》](https://nacos.io/zh-cn/docs/architecture.html)

