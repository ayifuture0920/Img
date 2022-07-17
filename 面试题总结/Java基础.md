## Java基础

### 1. 为什么Java代码可以实现一次编写、到处运行？

==参考答案==

JVM（Java虚拟机）是Java跨平台的关键。

在程序运行前，Java源代码（.java）需要经过编译器编译成字节码（.class）。在程序运行时，JVM负责将字节码翻译成特定平台下的机器码并运行，也就是说，只要在不同的平台上安装对应的JVM，就可以运行字节码文件。

**注意事项**

1. 编译的结果是生成字节码、不是机器码，字节码不能直接运行，必须通过JVM翻译成机器码才能运行；
2. 跨平台的是Java程序、而不是JVM，JVM是用C/C++开发的软件，不同平台下需要安装不同版本的JVM。

### 2.  一个Java文件里可以有多个类吗（不含内部类）？

==参考答案==

**参考答案**

1. 一个java文件里可以有多个类，但最多只能有一个被public修饰的类；
2. 如果这个java文件中包含public修饰的类，则这个类的名称必须和java文件名一致

### 3. 说一说你对Java访问权限的了解

==参考答案==

Java语言为我们提供了三种访问修饰符，即private、protected、public，在使用这些修饰符修饰目标时，一共可以形成四种访问权限，即private、default、protected、public，注意在不加任何修饰符时为default访问权限。

在修饰成员变量/成员方法时，该成员的四种访问权限的含义如下：

- private：该成员可以被该类内部成员访问；
- default：该成员可以被该类内部成员访问，也可以被同一包下其他的类访问；
- protected：该成员可以被该类内部成员访问，也可以被同一包下其他的类访问，还可以被它的子类访问；
- public：该成员可以被任意包下，任意类的成员进行访问。

在修饰类时，该类只有两种访问权限，对应的访问权限的含义如下：

- default：该类可以被同一包下其他的类访问；
- public：该类可以被任意包下，任意的类所访问。

### 4. Java中 Comparable 与 Comparator 的区别?

#### 相同

- Comparable和Comparator都是用来实现对象的比较、排序
- 要想对象比较、排序，都需要实现Comparable或Comparator接口
- Comparable和Comparator都是Java的接口

#### 区别

- Comparator位于java.util包下，而Comparable位于java.lang包下
- Comparable接口的实现是在类的内部（如 String、Integer已经实现了Comparable接口，自己就可以完成比较大小操作），Comparator接口的实现是在类的外部（可以理解为一个是自已完成比较，一个是外部程序实现比较）
- 实现Comparable接口要重写compareTo方法, 在compareTo方法里面实现比较

```java
public class Student implements Comparable {
     String name;
     int age
     public int compareTo(Student another) {
          int i = 0;
          i = name.compareTo(another.name); 
          if(i == 0) { 
               return age - another.age;
          } else {
               return i; 
          }
     }
}
   这时我们可以直接用 Collections.sort( StudentList ) 对其排序了.(
   **只需传入要排序的列表**）
```

- 实现 Comparator 需要重写 compare 方法

```java
public class Student{
     String name;
     int age
}
class StudentComparator implements Comparator { 
     public int compare(Student one, Student another) {
          int i = 0;
          i = one.name.compareTo(another.name); 
          if(i == 0) { 
               return one.age - another.age;
          } else {
               return i;          }
     }
}
   Collections.sort( StudentList , new StudentComparator()) 可以对其排序（
   **不仅要传入待排序的列表，还要传入实现了Comparator的类的对象**）
```