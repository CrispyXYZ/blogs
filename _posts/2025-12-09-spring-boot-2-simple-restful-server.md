---
layout: post
title: "Spring Boot(2)：简单 RESTful 服务器"
date: 2025-12-09 22:36:00 +0800
categories: [开发]
math: false
mermaid: false
pin: false
tags: [Java, Spring Boot, Controller]
---

项目地址：[https://github.com/crispyxyz/spring-demo](https://github.com/crispyxyz/spring-demo)

本文将在前文的基础上，实现一个简单的 RESTful 服务器。

## 第一个 RESTful 服务器

我们先来创建一个 `Book` 数据类：

```java
package io.github.crispyxyz.springdemo.model;

import lombok.Data;

@Data
public class Book {
    private String name;
    private String description;
}
```

然后，我们创建一个 `BookController` 类，用于处理 HTTP 请求：

```java
package io.github.crispyxyz.springdemo.controller;

import io.github.crispyxyz.springdemo.model.Book;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api")
public class BookController {
    private List<Book> books = new ArrayList<>();

    @PostMapping("/book")
    public ResponseEntity<List<Book>> addBook(@RequestBody Book book) {
        books.add(book);
        return ResponseEntity.ok(books);
    }

    @DeleteMapping("/book/{id}")
    public ResponseEntity<List<Book>> deleteBook(@PathVariable int id) {
        books.remove(id);
        return ResponseEntity.ok(books);
    }

    @GetMapping("/book")
    public ResponseEntity<List<Book>> getBookByName(@RequestParam String name) {
        return ResponseEntity.ok(books.stream().filter(book -> book.getName().equals(name)).collect(Collectors.toList()));
    }
}
```

这里，我们定义了四个 HTTP 方法：

- `POST /api/book`：用于添加书籍。
- `DELETE /api/book/{id}`：用于删除指定 ID 的书籍。
- `GET /api/book`：用于根据书名查询书籍。

此处有一些注解的使用：

- `@RequestBody`：用于将请求体中的 JSON 数据映射到 `Book` 对象。
- `@PathVariable`：用于获取路径参数。
- `@RequestParam`：用于获取查询参数。

运行，使用 Postman 测试，一切正常。

## 反思

注意到，上面的代码存在一定的问题：

- 书籍列表应该持久化到数据库中，而不是用一个简单的 `ArrayList` 来存储。（这个将在本系列后面的文章中介绍）
- 删除逻辑存在问题，应该根据 ID 而不是索引来删除。
- 缺少必要的 API ，比如获取所有书籍列表。
- 我们应该使用更加规范的 HTTP 状态码，比如 `201 Created`表示添加成功，`404 Not Found` 表示资源不存在。
- 我们应该使用更加严谨的异常处理，比如捕获 `IllegalArgumentException` 并返回 `400 Bad Request` 状态码。（这个将在本系列后面的文章中介绍）

## 改进

为了解决上面的部分问题，我们可以做以下改进：

### 完善 API

首先调整 API 结构，把 `/book` 加到类注解 `@RequestMapping("/api")` 里， 即：

```java
@RequestMapping("/api/books")
```

为了实现根据 ID 删除书籍，我们需要给 `Book` 类添加 `id` 字段，然后改用 `Map` 储存，并在 `BookController` 中添加一个 `deleteBookById` 方法：

```java
@Data
public class Book {
    private String id;
    private String name;
    private String description;
}
```

```java
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteBookById(@PathVariable String id) {
        if(books.remove(id) != null) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
```

接下来，是重新实现创建书籍的逻辑：

```java
    @PostMapping
    public ResponseEntity<Book> addBook(@RequestBody Book book) {
        String bookId = UUID.randomUUID().toString();
        book.setId(bookId);
        books.put(bookId, book);
        return ResponseEntity.created(URI.create("/api/books/" + bookId)).body(book);
    }
```

然后是 `GET` 方法：

```java
    @GetMapping
    public List<Book> getAllBooks() {
        return new ArrayList<>(books.values());
    }
```

注意这里返回值没有 `ResponseEntity` 包装，因为 Spring Boot 会自动包装成 `200 OK` 状态码的 `ResponseEntity`，其返回体为 `List<Book>` 的 JSON 字符串。

```java
    @GetMapping("/{id}")
    public ResponseEntity<Book> getBookById(@PathVariable String id) {
        if(books.containsKey(id)) {
            return ResponseEntity.ok(books.get(id));
        }
        return ResponseEntity.notFound().build();
    }
```

最后添加 `PUT` 方法：

```java
    @PutMapping("/{id}")
    public ResponseEntity<Book> updateBook(@PathVariable String id, @RequestBody Book updatedBook) {
        Book existingBook = books.get(id);
        if(existingBook == null) {
            return ResponseEntity.notFound().build();
        }

        if(updatedBook.getName() != null) {
            existingBook.setName(updatedBook.getName());
        }
        if(updatedBook.getDescription() != null) {
            existingBook.setDescription(updatedBook.getDescription());
        }

        return ResponseEntity.ok(existingBook);
    }
```

## 总结

本文介绍了 Spring Boot 如何实现一个简单的 RESTful 服务器，并对其进行了改进，使其更加符合 RESTful 规范。但仍然存在一些问题，将在后面的文章中解决。