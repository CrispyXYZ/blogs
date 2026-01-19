---
layout: post
title: "Java 作业第一阶段(4): 单元测试的实现"
date: 2026-01-19 19:30:00 +0800
categories: [开发]
math: false
mermaid: true
pin: false
tags: [Java, 面向对象, 单元测试, 作业]
---

本文延续上一节，继续完成该项目。

## 单元测试的实现

先找到 [相关资料](https://www.geeksforgeeks.org/advance-java/junit-executing-tests-with-maven-build/)。其详细介绍了如何在 Maven 项目中集成 JUnit 框架进行单元测试，包括项目配置、测试编写和测试执行的全过程。但此处我们选择较新版本的 JUnit 5 以使用 `@BeforeEach` 等新注解。

首先引入相关依赖和插件：

```xml
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.12.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.5.4</version>
            </plugin>
        </plugins>
    </build>
```

接着尝试编写一个测试类（参考 [该文章](https://liaoxuefeng.com/books/java/unit-test/junit-test/index.html)）：

```java
package io.github.crispyxyz.studentmanagement;

import io.github.crispyxyz.studentmanagement.exception.DataNotFoundException;
import io.github.crispyxyz.studentmanagement.model.Student;
import io.github.crispyxyz.studentmanagement.model.Teacher;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

import java.util.ArrayList;
import java.util.List;

public class GenericDataManagerTest {

    Student student1;
    Student student2;
    List<Student> studentList;

    @BeforeEach
    public void setUp() {
        student1 = new Student("1001", 18, "张三", "材料化学");
        student2 = new Student("1002", 20, "李四", "土木工程");
        studentList = new ArrayList<>();
    }

    @AfterEach
    public void tearDown() {
        student1 = null;
        student2 = null;
        studentList = null;
    }

    @Test
    public void testAdd() {
        GenericDataManager.add(studentList, student1);
        assertEquals(1, studentList.size());
        assertEquals(student1, studentList.get(0));

        GenericDataManager.add(studentList, student2);
        assertEquals(2, studentList.size());
        assertTrue(studentList.contains(student1) && studentList.contains(student2));
    }

    @Test
    public void testDeleteByIdSuccess() {
        GenericDataManager.add(studentList, student1);
        GenericDataManager.add(studentList, student2);

        try {
            GenericDataManager.deleteById(studentList, "1002");
        } catch(DataNotFoundException e) {
            fail();
        }
        assertEquals(1, studentList.size());
        assertTrue(studentList.contains(student1));
    }

    @Test
    public void testDeleteByIdNotFound() {
        GenericDataManager.add(studentList, student1);
        GenericDataManager.add(studentList, student2);

        DataNotFoundException exception = assertThrows(
            DataNotFoundException.class,
            () -> GenericDataManager.deleteById(studentList, "1003"));
        assertTrue(exception.getMessage().contains("1003"));
    }

    @Test
    public void testUpdateSuccess() {
        Student newStudent1 = new Student("1001", 19, "张三丰", "环境工程");

        GenericDataManager.add(studentList, student1);

        assertEquals(1, studentList.size());
        assertTrue(studentList.contains(student1));

        try {
            GenericDataManager.update(studentList, newStudent1);
        } catch(DataNotFoundException e) {
            fail();
        }

        assertEquals(1, studentList.size());
        assertTrue(studentList.contains(newStudent1));
    }

    @Test
    public void testUpdateNotFound() {
        Student newStudent = new Student("1003", 24, "李田所", "化学与应用化学");

        GenericDataManager.add(studentList, student1);

        DataNotFoundException exception = assertThrows(
            DataNotFoundException.class,
            () -> GenericDataManager.update(studentList, newStudent)
        );

        assertTrue(exception.getMessage().contains("1003"));
    }

    @Test
    public void testFindByIdSuccess() {
        GenericDataManager.add(studentList, student1);
        GenericDataManager.add(studentList, student2);

        Student foundStudent = null;
        try {
            foundStudent = GenericDataManager.findById(studentList, "1002");
        } catch(DataNotFoundException e) {
            fail();
        }

        assertEquals(student2, foundStudent);
    }

    @Test
    public void testFindByIdNotFound() {
        GenericDataManager.add(studentList, student1);

        DataNotFoundException exception = assertThrows(
            DataNotFoundException.class,
            () -> GenericDataManager.findById(studentList, "1002")
        );
        assertTrue(exception.getMessage().contains("1002"));
    }
}
```

运行，成功通过。累死，明天再更。