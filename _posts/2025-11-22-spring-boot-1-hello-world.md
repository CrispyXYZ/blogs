---
layout: post
title: "Spring Boot(1)：Hello World"
date: 2025-11-22 17:46:00 +0800
categories: [开发]
math: false
mermaid: false
pin: false
tags: [Java, Spring Boot, Controller, Web 开发, 后端开发, HTTP]
---

本系列用于记录从零开始学习 ~~（预习）~~ Spring Boot 的过程。

~~为什么不等到寒假时再来学呢，因为热爱qwq~~

## 项目结构

使用 IDEA 自带的选项创建一个 Spring Boot 项目，看一眼项目结构如下：

```
│  .gitattributes
│  .gitignore
│  HELP.md
│  mvnw
│  mvnw.cmd
│  pom.xml
│
├─.idea/...
│
├─.mvn
│  └─wrapper
│          maven-wrapper.properties
│
├─src
│  ├─main
│  │  ├─java
│  │  │  └─io
│  │  │      └─github
│  │  │          └─crispyxyz
│  │  │              └─springdemo
│  │  │                      SpringDemoApplication.java
│  │  │
│  │  └─resources
│  │      │  application.properties
│  │      │
│  │      ├─static
│  │      └─templates
│  └─test
│      └─java
│          └─io
│              └─github
│                  └─crispyxyz
│                      └─springdemo
│                              SpringDemoApplicationTests.java
│
└─target/...
```

~~好久没写 Java 了，怎么 Maven 也跟 Gradle 一样出了个 wrapper（~~

好熟悉的结构，这里着重了解一下不熟悉的资源文件部分：

- `application.properties`：Spring Boot 应用的核心配置文件，配置数据库连接、服务器端口、日志级别等 ~~（`properties` 文件对 Javaer 来说应该都不陌生了吧qwq）~~
- `static/`：存放静态资源文件，可通过浏览器直接访问
- `templates/`：存放页面模板文件，配合模板引擎使用，不能直接通过URL访问，需要通过控制器返回

再来看看主类 `SpringDemoApplication`：

```java
package io.github.crispyxyz.springdemo;

// import ...

@SpringBootApplication
public class SpringDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringDemoApplication.class, args);
    }
}
```

先运行试试：

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Web server failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

~~666怎么你用的也是8080端口~~

去配置文件里改改端口吧：

```properties
server.port=8888
```

运行：

```log
2025-11-22T17:04:22.329+08:00  INFO 25544 --- [spring-demo] [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8888 (http) with context path '/'
2025-11-22T17:04:22.334+08:00  INFO 25544 --- [spring-demo] [  restartedMain] i.g.c.springdemo.SpringDemoApplication   : Started SpringDemoApplication in 2.28 seconds (process running for 2.889)
```

访问看看：

> **Whitelabel Error Page**  
> This application has no explicit mapping for /error, so you are seeing this as a fallback.
> 
> Sat Nov 22 17:06:01 CST 2025  
> There was an unexpected error (type=Not Found, status=404).  
> No static resource .  
> org.springframework.web.servlet.resource.NoResourceFoundException: No static resource .

看来是能访问的，我们继续。

## 创建 `Controller`

~~一看到这个标题就知道这是 MVC 的思想了~~

使用 IDEA 自带的功能创建一个 REST 控制器类：

```java
package io.github.crispyxyz.springdemo.controller;

// import ...

@RestController
@RequestMapping("/")
class HelloWorldController {

}
```

停！先查阅一下这两个注解有什么功能：

- `RestController`：标记 REST API 控制器，控制器中的方法默认会将返回值直接作为 HTTP 响应体返回，而不是解析为视图名称，通常用于构建 RESTful API，支持返回 JSON、XML 等多种数据格式
- `RequestMapping`：定义请求映射路径，指定 URL 路径、 HTTP 方法等，将特定的 HTTP 请求路径和方法映射到对应的控制器方法进行处理，具体还有 `GetMapping`、`PostMapping` 等

我们可以尝试自己写写简单的 API ：

```java
@RestController
@RequestMapping("/spring-demo")
public class HelloWorldController {
    @GetMapping("/hello-world")
    public String helloWorld() {
        return "Hello, World!";
    }
}
```

运行看看效果：

![效果图](assets/img/spring-boot-1-screenshot.png)
