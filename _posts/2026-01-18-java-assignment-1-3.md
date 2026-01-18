---
layout: post
title: "Java 作业第一阶段(3): 多线程的实现"
date: 2026-01-18 15:14:00 +0800
categories: [开发]
math: false
mermaid: true
pin: false
tags: [Java, 面向对象, 多线程, 线程安全, 作业]
---

本文延续上一节，继续完成该项目。

## 多线程的实现

由于之前没怎么接触过多线程编程，故先查阅一下 [相关文章](https://www.cnblogs.com/xi-yongqi/p/19479394)。

得知，要保证线程安全，可以使用 Java 内置的 `synchronized` 关键字。其一共有 3 种使用方式，分别为：修饰实例方法（对象锁）、修饰静态方法（类锁），以及修饰代码块（自定义对象锁）。相关文章中讲的很明确，此处便不再赘述。

先实现一个线程不安全的 `enroll` 方法看看效果：

```java
    public boolean enroll(Student student) {
        if(remain<=0) return false;
        students.add(student);
        remain--;
        return true;
    }
```

抢课线程：

```java
public class EnrollThread extends Thread {
    private final Student student;
    private final Course course;

    public EnrollThread(Student student, Course course) {
        this.student = student;
        this.course = course;
    }

    @Override
    public void run() {
        try {
            Thread.sleep(1);
        } catch(InterruptedException e) {
            System.out.println("线程休眠被中断");
        }
        boolean result = course.enroll(student);
        System.out.printf("%s 抢课%s%n", student.getName(), result? "成功" : "失败");
    }
}
```

`main` 方法：

```java
    public static void main(String[] args) {
        List<Student> students = new ArrayList<>();
        List<Teacher> teachers = new ArrayList<>();

        Course javaCourse = new Course("CS001", "Java 从入门到入土", 5);

        GenericDataManager.add(students, new Student("001", 24, "甲", "计算机科学与技术"));
        GenericDataManager.add(students, new Student("002", 24, "乙", "计算机科学与技术"));
        GenericDataManager.add(students, new Student("003", 24, "丙", "计算机科学与技术"));
        GenericDataManager.add(students, new Student("004", 24, "丁", "计算机科学与技术"));
        GenericDataManager.add(students, new Student("005", 24, "戊", "计算机科学与技术"));
        GenericDataManager.add(students, new Student("006", 24, "己", "计算机科学与技术"));
        GenericDataManager.add(students, new Student("007", 24, "庚", "计算机科学与技术"));
        GenericDataManager.add(students, new Student("008", 24, "辛", "计算机科学与技术"));
        GenericDataManager.add(students, new Student("009", 24, "壬", "计算机科学与技术"));
        GenericDataManager.add(students, new Student("010", 24, "癸", "计算机科学与技术"));

        List<Thread> threads = new ArrayList<>();
        for(Student student : students) {
            threads.add(new EnrollThread(student, javaCourse));
        }

        for(Thread thread : threads) {
            thread.start();
        }

        try {
            Thread.sleep(500);
        } catch(InterruptedException e) {
            System.out.println("线程休眠被中断");
            System.exit(1);
        }

        System.out.println("抢课成功列表：");
        for(Student student : javaCourse.getStudents()) {
            System.out.println("- " + student.getName());
        }
        System.out.printf("选课人数：%d/%d%n", javaCourse.getCapacity() - javaCourse.getRemain(), javaCourse.getCapacity());
    }
```

运行输出：

```
[2026-01-18 14:44:07] [增] 影响对象ID: 001
[2026-01-18 14:44:07] [增] 影响对象ID: 002
[2026-01-18 14:44:07] [增] 影响对象ID: 003
[2026-01-18 14:44:07] [增] 影响对象ID: 004
[2026-01-18 14:44:07] [增] 影响对象ID: 005
[2026-01-18 14:44:07] [增] 影响对象ID: 006
[2026-01-18 14:44:07] [增] 影响对象ID: 007
[2026-01-18 14:44:07] [增] 影响对象ID: 008
[2026-01-18 14:44:07] [增] 影响对象ID: 009
[2026-01-18 14:44:07] [增] 影响对象ID: 010
癸 抢课成功
甲 抢课成功
丁 抢课成功
丙 抢课成功
乙 抢课成功
壬 抢课成功
己 抢课成功
辛 抢课失败
庚 抢课失败
戊 抢课失败
抢课成功列表：
- 己
选课人数：6/5

```

可见此时的 `enroll` 方法是线程不安全的，导致多个线程抢课时，出现数据混乱的情况。

## 线程安全的实现

继续着手解决线程安全问题。此处应该对 `enroll` 方法加上 `synchronized` 关键字，也就是对象锁。修改后运行看看（只截取关键部分）：

```
戊 抢课成功
己 抢课成功
乙 抢课成功
丁 抢课失败
丙 抢课成功
甲 抢课成功
壬 抢课失败
癸 抢课失败
庚 抢课失败
辛 抢课失败
抢课成功列表：
- 戊
- 甲
- 己
- 乙
- 丙
选课人数：5/5
```

完美！