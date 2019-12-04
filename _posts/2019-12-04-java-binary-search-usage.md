---
layout: post
title: Java中Collections的binarySearch方法| algorithom
category: JVM
tag: [JVM]
---

这篇文章主要记录了利用Java中Collections的binarySearch方法。
本文分为以下几个部分：
1. 方法一
2. 方法二


## Java中Collections的binarySearch方法

### 方法一
```
public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key)
```
此方法传入一个实现了Comparable接口的对象类的列表和要查找的元素。
创建实现了Comparable接口的对象类
```
public class Student1 implements Comparable<Student1> {

    private String name;
    private int age;

    public Student1() {
    }

    public Student1(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public int compareTo(Student1 s) {
        int num = this.age - s.age;
        int num1 = (num == 0 ? this.name.compareTo(s.name) : num);
        return num1;
    }
}
```

调用如下：
```
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import cn.stone.comparable_comparator.Student1;

public class Student1Test {
    public static void main(String[] args) {
        List<Student1> list1 = new ArrayList<Student1>();
        list1.add(new Student1("林青霞", 27));
        list1.add(new Student1("风清扬", 30));
        list1.add(new Student1("刘晓曲", 28));
        list1.add(new Student1("武鑫", 29));
        list1.add(new Student1("林青霞", 27));
        
        int index=Collections.binarySearch(list1, new Student1("林青霞", 27));
        System.out.println(index);
    }
}
```

### 方法二
```
public static <T> int binarySearch(List<? extends T> list, T key, Comparator<? super T> c)
```
此方法传入一个列表，要查找的元素，以及一个比较器。
创建对象类
```
public class Student2 {

    private String name;
    private int age;

    public Student2() {
    }

    public Student2(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```
调用:
```
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;
import cn.stone.comparable_comparator.Student2;

public class Student2Test {
    public static void main(String[] args) {
        List<Student2> list2 = new ArrayList<Student2>();
        list2.add(new Student2("林青霞", 27));
        list2.add(new Student2("风清扬", 30));
        list2.add(new Student2("刘晓曲", 28));
        list2.add(new Student2("武鑫", 29));
        list2.add(new Student2("林青霞", 27));

        int index1 = Collections.binarySearch(list2, new Student2("林青霞", 27),
                new MyComparator());
        System.out.println(index1);
    }
}

class MyComparator implements Comparator<Student2> {
    @Override
    public int compare(Student2 s1, Student2 s2) {
        int num = s1.getAge() - s2.getAge();
        int num1 = (num == 0 ? s1.getName().compareTo(s2.getName()) : num);
        return num1;
    }
}
```

注1：排序必须是升序

注2：方法二比较器也可采用匿名类实现