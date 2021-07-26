---
title: Nacos安装运行
date: 2021-07-26 13:30:51
tags: Nacos
---
## [快速开始](https://nacos.io/zh-cn/)

### Docker下载安装

``` bash
docker pull nacos/nacos-server
```

### Docker查看镜像

``` bash
docker images
```

### Docker运行

``` bash
docker run -d \
-e MODE=standalone \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=docker.for.mac.host.internal \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=123456 \
-e MYSQL_SERVICE_DB_NAME=nacos_config \
-p 8848:8848 \
--restart=always \
nacos/nacos-server
```

备注：连接数据库需要导入nacos自己的[数据库脚本](https://github.com/alibaba/nacos/edit/master/distribution/conf/nacos-mysql.sql)，该模式为单机启动

### Docker查看运行状态

``` bash
docker ps
```

### 访问地址

http://127.0.0.1:8848/  ，默认账密都是：nacos

