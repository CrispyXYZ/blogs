---
layout: post
title: "后端开发记录（1）：Pig 框架踩坑记录"
date: 2026-04-01 18:45:00 +0800
categories: [开发]
math: false
mermaid: true
pin: false
tags: [Java, Spring Boot, Spring Cloud, 微服务, 分布式, Pig 框架, Nacos, Maven, 后端开发]
---

本文（以及本系列文章）是本人初次基于 Pig 框架进行开发时，对一些有价值的问题进行的记录。

## 启动

### Docker 启动还是 IDE 启动？

一般在生产环境中，都是通过 Docker 启动。但是在 Windows 操作系统中，并不支持 Docker 环境。因此，如果在目前的开发阶段选择 Docker 启动，需要进行额外配置，且有潜在的网络问题、I/O性能受限等不便之处。所以，第一次启动，我选择在 IDE 内使用 Maven 启动。

### 启动 Nacos 失败？

先启动数据库，执行 `/db` 里的建表语句（修改数据库名称），在 Nacos 配置文件 `register` 模块中的 `resources/application.properties` 中修改密码，可能还需要解决端口冲突问题（因为 `8080`是常用端口）

### 启动其它服务时 Maven 报错？

```text
[ERROR] [ERROR] Some problems were encountered while processing the POMs:
[ERROR] Non-resolvable import POM: The following artifacts could not be resolved: com.pig4cloud:pig-common-bom:pom:3.9.2 (absent): Could not find artifact com.pig4cloud:pig-common-bom:pom:3.9.2 in aliyunmaven (https://maven.aliyun.com/repository/public) @ com.pig4cloud:pig:${revision}, D:\Projects\pig-learning\pig\pom.xml, line 107, column 16
[ERROR] 'dependencies.dependency.version' for com.pig4cloud:pig-common-core:jar is missing. @ com.pig4cloud:pig-gateway:${revision}, D:\Projects\pig-learning\pig\pig-gateway\pom.xml, line 57, column 15
[ERROR] 'dependencies.dependency.version' for org.springdoc:springdoc-openapi-starter-webflux-ui:jar is missing. @ com.pig4cloud:pig-gateway:${revision}, D:\Projects\pig-learning\pig\pig-gateway\pom.xml, line 62, column 15
[ERROR] 'dependencies.dependency.version' for com.github.xiaoymin:knife4j-openapi3-ui:jar is missing. @ com.pig4cloud:pig-gateway:${revision}, D:\Projects\pig-learning\pig\pig-gateway\pom.xml, line 67, column 15
[ERROR] 'dependencies.dependency.version' for cn.hutool:hutool-crypto:jar is missing. @ com.pig4cloud:pig-gateway:${revision}, D:\Projects\pig-learning\pig\pig-gateway\pom.xml, line 71, column 15
```

没有安装项目公用工具到本地仓库。应该先在根目录（`pig/`）执行 Maven 的 `install` 目标。

### 神秘错误： YAML 无法解析 @ 符号

这是由于占位符替换失败（通常是 Maven Profile 的变量），一般情况下不会遇到这个问题，如果你确信你的配置文件没有任何我错误，请尝试下面的解决方案。

#### 方案一：刷新本地缓存

执行：

```shell
mvn clean install
```

#### 方案二：指定 Maven Profile

尝试显式指定 profile，在 IDEA 中的 Maven 一栏的配置文件中手动勾选。