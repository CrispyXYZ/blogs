---
layout: post
title: "HTTP 通信(1)：简介与消息结构"
date: 2025-11-24 17:46:00 +0800
categories: [网络]
math: false
mermaid: false
pin: false
tags: [HTTP, TCP/IP, 包结构, 网络协议, 抓包]
---

本文将主要介绍一下 HTTP 通信。 HTTP 是一种基于 TCP/IP 模型的传输协议，定义了客户端与服务器之间请求和响应的格式。

目前广泛使用的版本是 HTTP/1.1 和 HTTP/2，也有正在逐步推广的 HTTP/3。其中 HTTP/1.1 和 HTTP/2 基于 TCP 协议，而 HTTP/3 基于 QUIC (UDP) 协议。

HTTP 消息分为两种类型：请求(Request)和响应(Response)。下面将以 HTTP/1.1 为例分别进行详解。在此版本中，TCP 数据包的 payload 部分即为 HTTP 消息数据。

可以参考 [RFC 9112](https://www.rfc-editor.org/rfc/rfc9112)。

## 请求消息

向服务端发送 GET 请求，抓包得到如下请求消息：

```http
GET /spring-demo/hello-world HTTP/1.1
Host: localhost:8888
Connection: keep-alive
Cache-Control: max-age=0
sec-ch-ua: "Chromium";v="142", "Microsoft Edge";v="142", "Not_A Brand";v="99"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36 Edg/142.0.0.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6

```

对其进行拆分，一个请求消息由以下部分组成：

```
请求行
请求头
空行
请求体（可选）
```

其中每个部分的都是使用 CRLF ( `\r\n` ) 换行。

|组成部分   |格式   |说明   |
|-         |-      |-      |
|请求行     |`请求方法␣URL␣协议版本`|`␣` = 空格|
|请求头     |`字段名:␣值` <br /> `字段名:␣值`|`␣` = 空格 <br /> 多个请求头需要 CRLF 换行分割|
|空行       |` `    |只含有 CRLF 换行|
|请求体（可选）| (数据) | 上面的示例是 GET 请求，没有这一部分|


## 响应消息

同样进行抓包：

```http
HTTP/1.1 200 
Content-Type: text/html;charset=UTF-8
Content-Length: 13
Date: Mon, 24 Nov 2025 13:14:02 GMT
Keep-Alive: timeout=60
Connection: keep-alive

Hello, World!
```

同样对其进行拆分，一个响应消息由以下部分组成：

```
状态行
响应头
空行
响应体（可选）
```

其中每个部分也是使用 CRLF ( `\r\n`，`0x0d 0x0a` ) 换行。

|组成部分   |格式   |说明   |
|-         |-      |-      |
|状态行     |`版本␣状态码␣状态信息（可选）`|`␣` = 空格|
|响应头     |`字段名:␣值` <br /> `字段名:␣值`|`␣` = 空格 <br /> 多个请求头需要 CRLF 换行分割|
|空行       |` `    |只含有 CRLF 换行|
|响应体（可选）| (数据) | 上面的示例是 GET 请求的响应，故有这一部分|