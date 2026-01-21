---
layout: post
title: "Java 作业第一阶段(5): 项目的完善"
date: 2026-01-21 20:16:00 +0800
categories: [开发]
math: false
mermaid: false
pin: false
tags: [Java, 面向对象, 作业]
---

上一节：[Java 作业第一阶段(4): 单元测试的实现]({% post_url 2026-01-20-java-assignment-1-4 %})

本文延续上一节，继续完成该项目。

## 项目的完善

项目走到此处，已经接近尾声了。再次回顾一下项目需求，还有以下内容尚未实现：

- `Teacher` 类中：只有在老师授课列表中的课程，才能被录入成绩。
- `main` 方法中：模拟管理员录入教师学生信息和学生课程成绩的过程，并对这些数据进行操作和展示。
- 注释的编写。

### 改进 `Teacher` 类

在 `Teacher` 类中，我们要实现只有在老师授课列表中的课程，才能被录入成绩。因此，新增一个 `recordScore` 方法：

```java
    public void recordScore(String course, Student student, double score) throws InvalidCourseException {
        if(!courses.contains(course)) {
            throw new InvalidCourseException("教师" + getName() + "不教授课程" + course + "，无法录入成绩");
        }
        ValidatingUtil.validateString(course, "课程");
        ValidatingUtil.validateDouble(score, 0.0, 100.0, "分数");
        student.getScores().put(course, score);
    }
```

对应的测试类此处不再展示。

### 完善 `main` 方法

此部分比较简单，直接扔 [commit `be2cc62`](https://github.com/CrispyXYZ/student-management/commit/be2cc62c8f8819e33b790496b7d884d7c666c59e)。

### 注释的编写

详见 [commit `1bea699`](https://github.com/CrispyXYZ/student-management/commit/1bea699f0be5d52941a9db74372253d600bea6d6)。

