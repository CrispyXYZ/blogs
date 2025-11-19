---
layout: post
title: "Java 复习(1)：Java基础复习规划"
date: 2025-11-18 22:56:00 +0800
categories: [语言]
math: false
mermaid: false
pin: false
tags: [Java, JVM, 多线程, 提纲]
---

~~本蒟蒻已经有至少3年没怎么写 Java 了，现列出复习提纲，希望找回当年的感觉qwq~~

## 核心知识梳理

### 数据类型与变量

- 8种基本数据类型：`byte`，`short`，`int`，`long`，`float`，`double`，`char`，`boolean`。掌握它们的取值范围、默认值、占用空间。
- 引用数据类型：类、接口、数组。理解引用与对象的区别。
- 自动装箱与拆箱：原理是什么？`Integer a = 100;` 发生了什么？注意 `-128~127` 的缓存问题（高频考点）。
- `==` 和 `equals()` 的区别：`==` 比较栈中的值（基本类型是数值，引用类型是地址），`equals()` 默认比较地址，但通常会被重写（如 `String`）。

### 面向对象（OOP）

- 四大特性： 封装、继承、多态、抽象。
- 重载 vs 重写： 区别是什么？（方法名、参数列表、返回值、异常、访问权限）
- 抽象类 vs 接口： 从Java 7、8、9三个版本来阐述区别。
  - Java 7: 抽象类有构造方法，可以有普通方法；接口只有抽象方法。
  - Java 8: 接口可以有 `default` 和 `static` 方法。
  - Java 9: 接口可以有 `private` 方法。
- 如何选择使用抽象类还是接口？（`is-a` 用抽象类，`has-a` 或行为契约用接口）
- `final` 关键字： 修饰变量、方法、类分别有什么作用？

### 核心类与常用API

- `String`，`StringBuffer`，`StringBuilder`:
  - `String` 的不可变性（底层 `final char[]`）。为什么设计为不可变？（安全、缓存、线程安全）
  - 三者在可变性、线程安全性、性能上的区别。
  - `String s = new String("abc");` 创建了几个对象？
- `Object` 类： 必须掌握的几个方法：
  - `equals()` 和 `hashCode()`： 为什么重写 `equals()` 必须重写 `hashCode()`？（在 HashMap 等集合中使用的约定）
  - `toString()`，`getClass()`，`clone()`，`wait()`/`notify()`/`notifyAll()`

### 异常处理

- `Throwable` 体系： `Error` vs `Exception`。
- 受检异常 vs 非受检异常： 区别、如何处理（`try-catch-finally`，`throws`）。
- `try-with-resources`: 原理（实现了 `AutoCloseable` 接口），优于传统的 `finally` 手动关闭。

### 集合框架

- 整体结构图： 能画出来 `Collection` 和 `Map` 两大体系。
- `List`:
  - `ArrayList`： 基于动态数组，查询快，增删慢。扩容机制（1.5倍）。
  - `LinkedList`： 基于双向链表，增删快，查询慢。
  - `Vector`： 线程安全的，但已过时。
- `Set`:
  - `HashSet`： 基于 `HashMap`，无序。
  - `LinkedHashSet`： 维护插入顺序。
  - `TreeSet`： 基于 `TreeMap`，有序。
- `Map`:
  - `HashMap`：
    - 底层结构： JDK 1.7：数组+链表；JDK 1.8：数组+链表/红黑树。
    - `put` 方法流程： 计算hash -> 找到数组下标 -> 遍历链表/红黑树 -> 插入。
    - 扩容机制： 何时触发？如何扩容？（2倍，rehash）
    - `hash()` 方法计算： 为什么用 `(h = key.hashCode()) ^ (h >>> 16)`？（扰动函数，减少哈希碰撞）
    - 线程不安全： 在并发下可能造成死循环（JDK 1.7）或数据覆盖。
  - `ConcurrentHashMap`： 如何保证线程安全？（JDK 1.7：分段锁；JDK 1.8：`synchronized` + CAS）
  - `HashTable`： 全表锁，性能差，已过时。
  - `LinkedHashMap`： 维护插入/访问顺序。
  - `TreeMap`： 基于红黑树，可排序。

### 输入输出（IO）

- 字节流 vs 字符流： `InputStream`/`OutputStream` vs `Reader`/`Writer`。
- 装饰者模式： 理解 `BufferedInputStream` 等包装流的原理。
- NIO (New IO)： 了解 `Channel`，`Buffer`，`Selector` 的概念，与BIO的区别（非阻塞、面向块）。

### 泛型

- 什么是类型擦除？编译后泛型信息还存在吗？
- `<? extends T>` 和 `<? super T>` 的区别（PECS原则）。

### 反射

- 什么是反射？`Class` 对象的获取方式（`.class`，`getClass()`，`Class.forName()`）。
- 优缺点：灵活性 vs 性能开销和安全性。

## 进阶深度理解

## Java内存模型（JMM）与垃圾回收（GC）

- JMM：
  - 内存区域：方法区、堆、栈、程序计数器、本地方法栈。
  - 堆内存结构：新生代（Eden，S0，S1）、老年代。
- GC：
  - 如何判断对象可回收？引用计数法 vs 可达性分析法。
  - 垃圾回收算法：标记-清除、标记-复制、标记-整理。
  - 常见的垃圾收集器：Serial，Parallel，CMS，G1，ZGC（了解特点和适用场景即可）。
- 类加载机制： 加载 -> 连接（验证、准备、解析）-> 初始化。双亲委派模型是什么？有什么好处？

### 多线程与并发

- 线程状态： `NEW`，`RUNNABLE`，`BLOCKED`，`WAITING`，`TIMED_WAITING`，`TERMINATED`。
- 创建线程的方式： 继承 `Thread` vs 实现 `Runnable`/`Callable`。
- 线程同步：
  - `synchronized`： 用法（修饰实例方法、静态方法、代码块）。底层原理（监视器锁monitor）。锁升级过程（无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁）。
  - `ReentrantLock`： 与 `synchronized` 的区别（可中断、公平锁、绑定多个条件）。
- `volatile` 关键字： 保证可见性和禁止指令重排序。不保证原子性。
- JUC (`java.util.concurrent`) 包：
  - `AtomicInteger`： CAS原理（Compare And Swap）。
  - `ThreadLocal`： 原理、内存泄露问题。
  - 线程池（必考）： `ThreadPoolExecutor` 的7大核心参数（核心线程数、最大线程数、存活时间、工作队列、线程工厂、拒绝策略）。工作流程。常见的线程池（`FixedThreadPool`，`CachedThreadPool`）及其潜在问题。

## 实战模拟与查漏补缺

### 手写代码

- 单例模式（懒汉式、饿汉式、DCL双重检查锁）。
- 生产者-消费者模型。
- 顺序打印ABC。
- 简单的算法题（排序、二分查找等）。

### 高频问题自测

- `HashMap` 和 `ConcurrentHashMap` 的原理区别？
- `synchronized` 和 `ReentrantLock` 的区别？
- `sleep()` 和 `wait()` 的区别？
- `ThreadLocal` 的原理和使用场景？
- 谈谈你对JVM内存模型和GC的理解。
- 什么是CAS？它有什么问题？（ABA问题）
- 谈谈类加载器和双亲委派模型。