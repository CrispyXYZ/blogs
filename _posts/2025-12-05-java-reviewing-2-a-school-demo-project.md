---
layout: post
title: "Java 复习(2)：Java 演示学校课程系统"
date: 2025-12-05 10:40:00 +0800
categories: [语言]
math: false
mermaid: false
pin: false
tags: [Java, 面向对象, 项目, 设计, Demo]
---

本文将使用 Java 演示学校课程系统的初步设计和实现。

## 设计

### 实体类

实体类包括：

- 课程类：课程编号、课程名称、开课教师。
- 学生类：学生编号、学生姓名、学生选课。
- 教师类：教师编号、教师姓名、教授课程。

### 管理类

管理类实现储存和管理实体类的功能。目前只有一个管理类，用于储存和管理所有实体类。

## 实现

### 实体类

#### 课程类

```java
package io.github.crispyxyz;

public final class Course {
    private final String id;
    private final String name;
    private final Teacher teacher;
    
    public Course(String id, String name, Teacher teacher) {
        this.id = id;
        this.name = name;
        this.teacher = teacher;
    }
    
    public String getId() {
        return id;
    }
    
    public String getName() {
        return name;
    }
    
    public Teacher getTeacher() {
        return teacher;
    }
}
```

#### 用户基类

```java
package io.github.crispyxyz;

public abstract class User {
    
    protected final int id;
    protected final String name;
    
    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }
    
    public abstract String getRole();
    
    public int getId() {
        return id;
    }
    
    public String getName() {
        return name;
    }
}
```

#### 学生类

```java
package io.github.crispyxyz;

import java.util.ArrayList;
import java.util.List;

public final class Student extends User {
    
    private final List<Course> courses = new ArrayList<>();
    
    public Student(int id, String name) {
        super(id, name);
    }
    
    public void addCourse(Course course) {
        courses.add(course);
    }
    
    public boolean removeCourse(Course course) {
        return courses.remove(course);
    }
    
    @Override
    public String getRole() {
        return "STUDENT";
    }
    
    public List<Course> getCourses() {
        return courses;
    }
    
}
```

#### 教师类

```java
package io.github.crispyxyz;

import java.util.ArrayList;
import java.util.List;

public final class Teacher extends User {
    
    private final List<Course> courses = new ArrayList<>();
    
    public Teacher(int id, String name) {
        super(id, name);
    }
    
    public void addCourse(Course course) {
        courses.add(course);
    }
    
    public boolean removeCourse(Course course) {
        return courses.remove(course);
    }
    
    @Override
    public String getRole() {
        return "TEACHER";
    }
    
    public List<Course> getCourses() {
        return courses;
    }
    
}
```

### 管理类

```java
package io.github.crispyxyz;

import java.util.ArrayList;
import java.util.List;

public final class SchoolManager {
    private final List<Student> students = new ArrayList<>();
    private final List<Teacher> teachers = new ArrayList<>();
    private final List<Course> courses = new ArrayList<>();

    public void addStudent(Student student) {
        students.add(student);
    }

    public void addTeacher(Teacher teacher) {
        teachers.add(teacher);
    }

    public void addCourse(Course course) {
        courses.add(course);
    }

    public void removeStudent(Student student) {
        students.remove(student);
    }

    public void removeTeacher(Teacher teacher) {
        teachers.remove(teacher);
    }

    public void removeCourse(Course course) {
        courses.remove(course);
    }

    public List<Student> getStudents() {
        return students;
    }

    public List<Teacher> getTeachers() {
        return teachers;
    }

    public List<Course> getCourses() {
        return courses;
    }
}
```

## 运行

```java
package io.github.crispyxyz;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        SchoolManager manager = new SchoolManager();
        Student stu1 = new Student(1, "张三");
        Student stu2 = new Student(2, "李四");
        Teacher tea1 = new Teacher(3, "王五");
        Teacher tea2 = new Teacher(4, "赵C");
        Course course1 = new Course("MA01", "数学分析", tea1);
        Course course2 = new Course("MA02", "解析几何", tea2);
        stu1.addCourse(course1);
        stu1.addCourse(course2);
        stu2.addCourse(course2);
        tea1.addCourse(course1);
        tea2.addCourse(course2);

        manager.addStudent(stu1);
        manager.addStudent(stu2);
        manager.addTeacher(tea1);
        manager.addTeacher(tea2);
        manager.addCourse(course1);
        manager.addCourse(course2);

        manager.getStudents().stream().map(stu -> String.join("", List.of(String.valueOf(stu.getId()), "号 ", stu.getName(), ": ", String.join(", ", stu.getCourses().stream().map(course -> String.join("-", course.getId(), course.getName(), course.getTeacher().getName())).collect(Collectors.toCollection(ArrayList::new)))))).forEach(System.out::println);
    }
}
```

输出：

```
1号 张三: MA01-数学分析-王五, MA02-解析几何-赵C
2号 李四: MA02-解析几何-赵C
```

## 总结

本文使用 Java 演示了学校课程系统的设计和实现。实体类包括课程类、学生类、教师类，管理类实现储存和管理实体类的功能。

该实现的不足之处主要在于系统架构与功能完整性方面：其系统扩展性受限，关键实体类被标记为 `final` 且管理类直接暴露了内部集合，破坏了封装；同时，数据仅存在于内存，缺乏持久化机制，核心业务逻辑如选课约束也未实现；此外，代码缺少必要的异常处理，部分实现可读性较差。整体来看，当前版本更偏向于一个演示核心关系的骨架模型，离一个健壮、可用的完整系统尚有距离。