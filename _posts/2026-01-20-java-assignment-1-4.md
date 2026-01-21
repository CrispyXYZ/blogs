---
layout: post
title: "Java 作业第一阶段(4): 单元测试的实现"
date: 2026-01-20 22:43:00 +0800
categories: [开发]
math: false
mermaid: true
pin: false
tags: [Java, 面向对象, 测试, JUnit, 作业]
---

上一节：[Java 作业第一阶段(3): 多线程的实现]({% post_url 2026-01-18-java-assignment-1-3 %})

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

运行，成功通过。

接下来是 `ScoreManager` 类的测试：

```java
package io.github.crispyxyz.studentmanagement;

import io.github.crispyxyz.studentmanagement.model.Student;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class ScoreManagerTest {

    List<Student> studentList;
    Student student1;
    Student student2;
    Student student3;

    @BeforeEach
    public void setUp() {
        studentList = new ArrayList<>();

        Map<String, Double> scores1 = new HashMap<>();
        scores1.put("军事理论", 97.0);
        scores1.put("微积分(2)", 59.9);

        student1 = new Student("1001", 19, "张三", "生物工程");
        student1.setScores(scores1);
        studentList.add(student1);

        Map<String, Double> scores2 = new HashMap<>();
        scores2.put("军事理论", 59.7);
        scores2.put("微积分(2)", 47.9);

        student2 = new Student("1002", 20, "李四", "应用化学");
        student2.setScores(scores2);
        studentList.add(student2);

        Map<String, Double> scores3 = new HashMap<>();
        scores3.put("军事理论", 100.0);
        scores3.put("微积分(2)", 100.0);

        student3 = new Student("1003", 24, "王五", "材料物理");
        student3.setScores(scores3);
        studentList.add(student3);
    }

    @AfterEach
    public void tearDown() {
        studentList = null;
        student1 = null;
        student2 = null;
        student3 = null;
    }

    @Test
    public void testGetCourseStat() {
        ScoreManager.CourseStat stat = ScoreManager.getCourseStat(studentList, "军事理论");
        assertEquals(2567.0/30.0, stat.average, 0.001);
        assertEquals(2.0/3.0, stat.passRate, 0.001);
    }

    @Test
    public void testGetFailedStudents() {
        Map<Student, Integer> failedStudents = ScoreManager.getFailedStudents(studentList);

        assertTrue(failedStudents.containsKey(student1));
        assertTrue(failedStudents.containsKey(student2));
        assertFalse(failedStudents.containsKey(student3));

        assertEquals(1, failedStudents.get(student1));
        assertEquals(2, failedStudents.get(student2));
    }
}
```

运行，发现测试失败：

```
org.opentest4j.AssertionFailedError: 
预期:85.56666666666666
实际:85.33333333333333
```

我们回头检查一下 `ScoreManager.getCourseStat` 方法。发现我们使用了 `int scoreSum` 用于储存总分数。我们将其改为 `double scoreSum` 即可解决此问题。

剩下的部分……直接扔 [commit](https://github.com/CrispyXYZ/student-management/commit/a6b45f40b174dfd4c6dd6b489b149d022c07b7a6) 吧，敲代码敲入迷了。

下一节：[Java 作业第一阶段(5): 项目的完善]({% post_url 2026-01-21-java-assignment-1-5 %})