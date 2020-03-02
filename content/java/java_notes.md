---
title: "Java Notes(quick guide for C++ programmers)"
date: 2019-07-20 01:01
tag: java
---

[TOC]

我没有系统学过 Java，从一个 C++ 程序员的角度简单梳理下 Java 语言的基本特性，目的是能上手写一些简单程序。

## 用 Eclipse 编写 Java 代码

快捷键：

- ALT + / ：自动补全
- CTRL + F11 ： 运行当前 Java 文件
- CTRL+SHIFT+F：格式化代码

---

## Java 基础语法

### 数据类型

Java 中有两大数据类型：

- simple Java data，基本类型，包括 boolean, character, integer, real number
- Java objects，即 Java 中的“对象”

一个 C++ 和 Java 的小区别，假设 x 是 int 型变量，那么

```cpp
while (x) {
    ......
}
```

在 C++ 中是正确的，但是在 Java 中一定要写成 `while (x != 0)`。

对于 Java 的基本数据类型，赋值操作（assignment，即 `=` 操作）是按值拷贝的。但对于 Java objects，`=` 这个操作是“浅拷贝”，即相当于引用。（关于浅拷贝和深拷贝，这里只要记住 `=` 不能完成深拷贝，只相当于传递了对象的引用）。

### Java 中的 Messages

消息是 OOP 中的核心概念。Java 中的表达形式：

```java
receiver.method-name(parameter list)
```

或者

```java
receiver.variable-name
```

这里 `receiver` 可以是 class 或 object。

例子：

```java
System.out.println(x.getClass().getName());
```

### 创建 Java 中的对象

Java class 的构造函数形式：

```java
public class-name(typed parameter list) {
        // object initialization code
}
```

函数名是类名，没有返回值

构造函数调用方式（创建新的实例对象）：

```java
new class-name(parameter list)
```

除了创建实例，构造函数还有可能有如下调用方式：

```java
this(parameter list);  // 调用该类重载的构造函数
super(parameter list);  // 超类的构造函数
```

例子：

```java
public String() {
     this("");
}
```

### Java 类的定义

```java
public class A {
     object and class member definitions
}
```

和 C++ 类似，Java 中也有 实例成员（instance member） 和 类成员（class member） 的区别。

用 static 关键字声明的成员是类成员。

### Java 中的常量（constants）

Java 中用 final 关键字声明 constants：

```java
static final int taxRates;
```

### Java 中的访问域（access scope）

和 C++ 类似有 public, private 和 protected

有一点要注意，Java 中有 “package scope”，其范围为一个包含 java source code files 的目录，相同的 package scope 的 java 源文件都包含：

```java
package package-name;
```

### Java 中的 null 变量

null 可以赋值给任何对象，比如赋值给一个 list 对象，表示 list 对象为空。

### Java 中的继承

用 extends 关键字：

```java
public class B
    extends A {
     instance and class member definitions
}
```

### Java 中的抽象类

使用 abstract 关键字：

```java
public abstract class A {
     member definitions
     public abstract boolean func(int n);
     more member definitions
}

public class B
    extends A {
     new member definitions
     public boolean func(int n) {
         implementation code
     }
     more new member definitions
}
```

### Java 中的接口（interface）

interface 和 抽象类类似，但是其中所有方法都必须是空的，而且如果有数据成员，必须用 final 声明。

接口通过 implements 实现。

```java
public interface C {
     public boolean func(int n);
     more method declarations
}

public class E
    extends A
    implements C, D {
     definitions of members declared in A, C, and D
     new member definitions
}
```

### Java 中的数组（Arrays）与字符串（Strings）

数组：

```java
int [] A = new int[5];
A[2] = 3;
System.out.println("The value of A[2] is " + A[2]);
```

注意这里自动进行了 int 转 String 。

Java 中的字符串不像 C 中那样是 char 类型的数组，一般通过类 String 表示。

Java 中的所有对象都有一个 toString() 方法。

### Java 中的通用数据结构和强制转换

常用数据结构：

- Hashtable
- Vector
- Stack

接口类型：

- Enumeration

强制转换：

```java
(String)(enum.nextElement())
```

### Java 源代码和 package 命名规则

一般情况下，一个 Java 源文件中应该只定义一个 class，文件名就应该是类名，比如 Comparators.java 。

好的编程规范是将一组相关的 java 源文件写到同一个 package 中。每个 package 应该单独占一个文件夹。

注意 package 可以有层级关系，组织文件时按照子文件夹来组织，比如

```java
package ig.util;  // ig 包下有一个 util 包
```

文件夹形式应该为：

```
/ig/util
```

### Import 语句

```java
import fully-qualified-class-name;
```

注意 fully-qualified-class-name 的形式应该为：

```java
package-name.class-name
```

例子：

```java
import ig.util.Comparators;
import java.util.*;
```

### Java 源文件的结构

一个 .java 文件应该有如下结构：

```java
package statement
import statements
class definition
```

### Java 文件的编译与执行

```
javac A.java
java A
```

## Java 程序库

### Java 中的常用“容器”

基本接口：

- Set
- List
- Map

常用实现：

- ArrayList
- LinkedList
- HashSet
- HashMap
- LinkedHashSet
- LinkedHashMap
- TreeSet
- TreeMap

常用方法：

```java
public boolean isEmpty()

public boolean contains(Object obj)

public int size()

public boolean equals(Object other)

public Object[] toArray()
```

### ArrayList

直接看一个例子：

```java
import java.util.ArrayList;

public class test {
     public static void main(String[] args) {
         
          // 创建 ArrayList
          ArrayList<String> stringList = new ArrayList<String>();
         
          // 添加元素，在尾部添加新元素
          stringList.add("foo");
          stringList.add("bar");
          stringList.add("zot");
          stringList.add("bah");
         
          // 使用 get() 获取指定下标元素
          System.out.println("Size of List: " + stringList.size());
          System.out.println("Contents: ");
          for (int i = 0; i < stringList.size(); ++i) {
               System.out.println("At index " + i + " value = " + stringList.get(i));
          }
          System.out.println();
         
          // 删除元素，可以按值或按下标
          stringList.remove("bar");
          System.out.println("Contents after remove \"bar\": ");
          for (int i = 0; i < stringList.size(); ++i) {
               System.out.println("At index " + i + " value = " + stringList.get(i));
          }
          System.out.println();
         
          stringList.remove(1);
          System.out.println("Contents after remove element of index 1: ");
          for (int i = 0; i < stringList.size(); ++i)
          {
               System.out.println("At index " + i + " value = " + stringList.get(i));
          }
          System.out.println();
         
          // 在指定位置添加元素
          stringList.add(1, "meh");
          System.out.println("Contents after add \"meh\" at index 1: ");
          for (int i = 0; i < stringList.size(); ++i)
          {
               System.out.println("At index " + i + " value = " + stringList.get(i));
          }
          System.out.println();
         
          // 设置指定位置的元素值
          stringList.set(1, "bar");
          System.out.println("Contents after set \"bar\" at index 1: ");
          for (int i = 0; i < stringList.size(); ++i)
          {
               System.out.println("At index " + i + " value = " + stringList.get(i));
          }
          System.out.println();
         
          // 查询元素下标
          System.out.println("Index of \"bar\": ");
          System.out.println(stringList.indexOf("bar") + "\n");
         
          // 查询失败返回 -1
          System.out.println("Index of \"zzz\": ");
          System.out.println(stringList.indexOf("zzz") + "\n");
     }
}

```

注意 indexOf() 方法基于容器内元素类型的 equals() 方法，如果没有定义 equals() 方法，indexOf() 总会返回 -1。

下面是一个自定义元素类型的例子：

```java
// MyIntClass.java
// Simple integer class with just two constructors.
// A more robust version would make the int m_value private with
// accessor methods instead
public class MyIntClass {
    
     public int value;
    
     public MyIntClass() {
          value = 0;
     }
    
     public MyIntClass(int val) {
          value = val;
     }
    
     public boolean equals(Object otherIntObject) {
          MyIntClass otherInt = (MyIntClass) otherIntObject;
          if (this.value == otherInt.value)
               return true;
         
          return false;
     }
}
```

```java
import java.util.ArrayList;
public class test {
     public static void main(String[] args) {
          ArrayList<MyIntClass> arr = new ArrayList<MyIntClass>();
         
          for (int i = 0; i < 4; ++i) {
               arr.add(new MyIntClass(i));
          }
          System.out.println("Size: " + arr.size() + "\n");
         
          System.out.println("Original ArrayList: ");
          PrintArrayList(arr);
         
          arr.remove(2);
          System.out.println("After remove: ");
          PrintArrayList(arr);
         
          arr.add(1, new MyIntClass(100));
          System.out.println("After insert :");
          PrintArrayList(arr);
         
          // 如果 MyIntClass 中没有 equals() 方法，会返回 -1
          System.out.println("Position of 1");
          System.out.println(arr.indexOf(new MyIntClass(1)));
     }
     public static void PrintArrayList(ArrayList<MyIntClass> arr) {
          MyIntClass tmp;
         
          for (int i = 0; i < arr.size(); ++i) {
               tmp = arr.get(i);
               System.out.println(tmp.value);
          }
          System.out.println();
     }
}
```

#### 迭代器形式遍历 ArrayList

```java
for (BaseType varName : ArrayListVariable) {
  Statement with varName
}
```

```java
public static void PrintArrayList(ArrayList<MyIntClass> arr) {
 
  for (MyIntClass intObj : arr) {
       System.out.println(intObj.value);
  }
  System.out.println();
}
```

### HashSet

HashSet 是基于 Set 接口的一个集合类实现（底层通过 hash table 实现），其中保证没有重复元素。所以 HashSet 是一个集合类容器，不是哈希表，注意其中的元素是无序的。如果需要有序的集合，要使用 TreeSet。直接看例子代码：

```java
import java.util.HashSet;
import java.util.Iterator;
public class test {
     private static void outputSet(HashSet<String> set) {
          for (String s : set) {
               System.out.println(s + " ");
          }
          System.out.println();
     }
    
     public static void main(String[] args) {
          HashSet<String> round = new HashSet<String>();
          HashSet<String> green = new HashSet<String>();
         
          round.add("peas");
          round.add("ball");
          round.add("pie");
          round.add("grapes");
         
          green.add("peas");
          green.add("grapes");
          green.add("garden hose");
          green.add("grass");
         
          System.out.println("Contents of set round: ");
          outputSet(round);
         
          System.out.println("\nContents of set green: ");
          outputSet(green);
         
          // contains() 方法检查是否存在某一元素
          System.out.println("\nball and peas in same set? " +
                    ((round.contains("ball") &&
                    (round.contains("peas"))) ||
                    (green.contains("ball") &&
                    (green.contains("peas")))));
         
          System.out.println("\npie and grass in same set? " +
                    ((round.contains("pie") &&
                    (round.contains("grass"))) ||
                    (green.contains("pie") &&
                    (green.contains("grass")))));
         
          // 利用 addAll() 方法求两个 HashSet 的并集
          HashSet<String> setUnion = new HashSet<String>(round);
          setUnion.addAll(green);
          System.out.println("\nUnion of green and round: ");
          outputSet(setUnion);
         
          // 利用 removeAll() 方法求两个 HashSet 的交集
          HashSet<String> tmp = new HashSet<String>(round);
          tmp.removeAll(green);
          HashSet<String> setInter = new HashSet<String>(round);
          setInter.removeAll(tmp);
          System.out.println("\nIntersection of green and round: ");
          outputSet(setInter);
         
     }
}
```

要在 HashSet 中使用自定义类型，需要实现自定义类的 hashCode() 和 equals() 方法。

### Map 和 HashMap

注意不要被上面 HaseSet 迷惑，Map 才是保存 key-value 对的容器。要保证 key 的类型有 hashCode() 和 equals() 方法。

看例子代码：

```java
import java.util.HashMap;
import java.util.Scanner;
public class test {
    
     public static void main(String[] args) {
          HashMap<Integer, String> employees = new HashMap<Integer, String>(10);
         
          employees.put(10,  "Joe");
          employees.put(49,  "Andy");
          employees.put(91, "Greg");
          employees.put(70, "Kiki");
          employees.put(99, "Antoinette");
          System.out.print("Added Joe, Andy, Greg, Kiki, ");
          System.out.println("and Antoinette to the map.");
         
          System.out.println("The map contains: ");
          for (Integer key : employees.keySet()) {
               System.out.println(key + " : " + employees.get(key));    
          }
          System.out.println();
         
          // 从键盘输入 id
          Scanner keyboard = new Scanner(System.in);
          int id;
          do {
               System.out.print("\nEnter an id to look up in the map. ");
               System.out.println("Enter -1 to quit.");
               id = keyboard.nextInt();
               if (employees.containsKey(id)) {
                    String e = employees.get(id);
                    System.out.println("ID found: " + e.toString());
               } else if (id != -1)
                    System.out.println("ID not found.");
          } while (id != -1);
     }
}
```

---

## Reference

[c2java](https://www.d.umn.edu/~gshute/java/c2java.html)

