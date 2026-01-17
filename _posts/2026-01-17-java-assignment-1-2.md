---
layout: post
title: "Java 作业第一阶段(2): 基本功能的实现"
date: 2026-01-17 19:49:00 +0800
categories: [开发]
math: false
mermaid: true
pin: false
tags: [Java, 面向对象, 泛型, 异常处理, 日志, CRUD, 作业]
---

上一节：[Java 作业第一阶段(1): 实体类的实现]({% post_url 2026-01-16-java-assignment-1-1 %})

本文延续上一节，继续完成该项目。

## 基本功能的实现

~~看到泛型第一反应 `template <typename T>` 是不是已经没救了（~~

总感觉 `GenericDataManager` 的需求讲的不是很明确，不过还是先写写看吧 ~~（摸着石头过河）~~ 。类比 Java 集合框架，可以设计一个泛型类 `GenericDataManager<T extends Person>`，内部维护一个 `List<T>` 储存数据。

然后就出问题了：这时发现 `ScoreManager` 需要访问学生和成绩的的列表信息。所以之前的泛型类的设计会过于复杂：需要 `getter` 方法来获取内部的 `List<T>`，再传给 `ScoreManager`。于是重新设计 `GenericDataManager`，在外部（例如 `main` 函数）维护各种 `List`，以函数参数的形式传递。由此，可以将两个 `Manager` 设计为只提供静态方法的类。

继续设计 `ScoreManager`。统计某一课程的成绩情况，需要一次返回多个数据。这让我想到了 C/C++ 的 `struct` 结构体用于储存数据。遂编写一个 `CourseStat` 类，内有两个 `public final` 的属性 `average` 和 `passRate` 及其构造函数（其实也可以用 JDK 较新版本中的 `record`，不过此处为兼容 JDK 8 故不使用 `record`）。

下面进行异常处理。在构造实体类的实体对象时，需要注意参数的合法性。在处理此问题时，通常在 `setter` 方法中用 `if` 语句判断。由此，构造方法中 `this.foo=foo` 的写法也需要修改。故将其改为调用 `setter`。此处也增加了一个自定义异常类 `DataNotFoundException`，用于处理 `id` 不存在的情况。

最后是日志记录。一开始还没看懂 `Object data` 这个参数的用意，先随便按我的理解写了写 `log` 方法。然后去 CRUD 部分加 log 时，才发现有的方法的参数是 `Person` 给的 `id`，有的直接就是 `String`。于是茅塞顿开，最终完成了基本功能的实现。