---
layout: post
title: "Java 复习(3)：改进管理系统"
date: 2025-12-14 22:56:00 +0800
categories: [语言]
math: false
mermaid: false
pin: false
tags: [Java, 面向对象, 项目, Lombok, 单例模式, 工厂方法, 建造者模式, 线程安全]
---

上一届使用 Java 演示了学校课程系统的初步设计和实现。本文将对管理系统进行改进，提升系统的可用性和可靠性。

## 改进过程

### 引入 Lombok 库

先引入 Lombok 库：

```xml
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.42</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

然后利用 Lombok 改进实体类，包括使用 `@Getter` 注解简化代码，以及修改访问权限修饰符，以供后续管理类的重构，同时调整包名。

### `User` 类

主要是引入 Lombok 的 `@SuperBuilder` 和 `@RequiredArgsConstructor` 注解：

```java
package io.github.crispyxyz.schoolmanager;

import lombok.Getter;
import lombok.RequiredArgsConstructor;
import lombok.experimental.SuperBuilder;

@Getter
@SuperBuilder
@RequiredArgsConstructor(access = lombok.AccessLevel.PROTECTED)
public abstract class User {
    protected final int id;
    protected final String name;

    public abstract String getRole();
}
```

### `Course` 类

引入 Lombok 注解，并增加学生列表的双向关联：

```java
package io.github.crispyxyz.schoolmanager;

import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.experimental.SuperBuilder;

import java.util.ArrayList;
import java.util.List;

@Getter
@SuperBuilder
@AllArgsConstructor(access = AccessLevel.PACKAGE)
public final class Course {
    private final String id;
    private final String name;
    private final Teacher teacher;
    private final List<Student> students = new ArrayList<>();

    void addStudent(Student student) {
        students.add(student);
    }

    boolean removeStudent(Student student) {
        return students.remove(student);
    }

    public List<Student> getStudents() {
        return List.copyOf(students);
    }
}
```

### `Teacher` 类

用 Lombok 的 `@SuperBuilder` 实现了建造者模式，同时将构造函数设为包私有：

```java
package io.github.crispyxyz.schoolmanager;

import lombok.Getter;
import lombok.experimental.SuperBuilder;

import java.util.ArrayList;
import java.util.List;

@Getter
@SuperBuilder
public final class Teacher extends User {
    
    private final List<Course> courses = new ArrayList<>();
    
    Teacher(int id, String name) {
        super(id, name);
    }
    
    void addCourse(Course course) {
        courses.add(course);
    }
    
    boolean removeCourse(Course course) {
        return courses.remove(course);
    }
    
    @Override
    public String getRole() {
        return "TEACHER";
    }

    public List<Course> getCourses() {
        return List.copyOf(courses);
    }
}
```

### `Student` 类

与 `Teacher` 类类似，引入 Lombok 的 `@SuperBuilder` 注解，并修改构造函数为包私有：

```java
package io.github.crispyxyz.schoolmanager;

import lombok.Getter;
import lombok.experimental.SuperBuilder;

import java.util.ArrayList;
import java.util.List;

@Getter
@SuperBuilder
public class Student extends User {
    private final List<Course> courses = new ArrayList<>();

    Student(int id, String name) {
        super(id, name);
    }

    void addCourse(Course course) {
        courses.add(course);
    }

    boolean removeCourse(Course course) {
        return courses.remove(course);
    }

    @Override
    public String getRole() {
        return "STUDENT";
    }

    public List<Course> getCourses() {
        return List.copyOf(courses);
    }
}
```

### `SchoolManager` 类

通过单例模式和工厂方法重构：

```java
package io.github.crispyxyz.schoolmanager;

import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public final class SchoolManager {
    private final Map<Integer, Student> students = new ConcurrentHashMap<>();
    private final Map<Integer, Teacher> teachers = new ConcurrentHashMap<>();
    private final Map<String, Course> courses = new ConcurrentHashMap<>();

    private SchoolManager() {}

    private static final SchoolManager INSTANCE = new SchoolManager();

    public static SchoolManager getInstance() {
        return INSTANCE;
    }

    public Student createStudent(String name) {
        int id = students.size();
        Student student = Student.builder()
                .id(id)
                .name(name)
                .build();

        students.put(id, student);
        return student;
    }

    public Teacher createTeacher(String name) {
        int id = teachers.size();
        Teacher teacher = Teacher.builder()
                .id(id)
                .name(name)
                .build();

        teachers.put(id, teacher);
        return teacher;
    }

    public Course createCourse(String id, String name, Teacher teacher) {
        Course course = Course.builder()
                .id(id)
                .name(name)
                .teacher(teacher)
                .build();
        courses.put(id, course);
        teacher.addCourse(course);
        return course;
    }

    public void enrollStudentInCourse(Student student, Course course) {
        student.addCourse(course);
        course.addStudent(student);
    }

    public List<Student> getStudents() {
        return List.copyOf(students.values());
    }
}
```

### `Main` 类

```java
package io.github.crispyxyz.schoolmanager.app;

import io.github.crispyxyz.schoolmanager.Course;
import io.github.crispyxyz.schoolmanager.SchoolManager;
import io.github.crispyxyz.schoolmanager.Student;
import io.github.crispyxyz.schoolmanager.Teacher;

public class Main {
    public static void main(String[] args) {
        SchoolManager manager = SchoolManager.getInstance();

        Student zhangSan = manager.createStudent("张三");
        Student liSi = manager.createStudent("李四");
        Student wangWu = manager.createStudent("王五");

        Teacher mcArthur = manager.createTeacher("麦克阿瑟");
        Teacher chiangKaishek = manager.createTeacher("常凯申");

        Course militaryTheoryA = manager.createCourse("MT-A", "军事理论A", mcArthur);
        Course militaryTheoryB = manager.createCourse("MT-B", "军事理论B", chiangKaishek);
        Course militaryLogisticsA = manager.createCourse("ML-A", "军事物流学A", chiangKaishek);

        manager.enrollStudentInCourse(zhangSan, militaryTheoryA);
        manager.enrollStudentInCourse(liSi, militaryTheoryB);
        manager.enrollStudentInCourse(wangWu, militaryTheoryA);
        manager.enrollStudentInCourse(wangWu, militaryLogisticsA);

        manager.getStudents().forEach(student -> {
            System.out.println(student.getId() + ". " + student.getName() + ": ");
            student.getCourses()
            .forEach(course -> System.out.println(course.getId() + " " + course.getName() + ", " + course.getTeacher()
            .getId() + " " + course.getTeacher().getName()));
        });
    }
}
```

## 总结

改进版本成功运用 `lombok` 库简化了代码结构，通过单例模式和工厂方法统一了对象创建逻辑，增强了系统的封装性和可维护性，同时利用 `ConcurrentHashMap` 保障了基础线程安全。

然而，该版本仍存在 ID 生成策略缺陷（使用集合大小可能导致重复ID）、异常处理机制缺失、数据一致性保障不足（如删除对象时未清理关联关系）以及业务功能不完整（缺少查询、更新等核心操作）等问题，这些都有待进一步优化以提升系统的健壮性和实用性。