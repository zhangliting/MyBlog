---
title: JVM 内存模型
tags: [JVM]
categories: [JVM]
photo:
  - /images/jvm_overview.jpg
date: 2017-04-24 20:56:35
---
为了实现Java最重要的思想：`一次编写，到处运行`。`Sun`公司创建了Java虚拟机。虚拟机抽象了底层的操作系统，解析编译后的Java代码。JVM（Java Virtual Machine）是JRE（Java Runtime Environment）的核心，用于运行Java代码，现在也被其他语言使用（Scala，Groovy，JRuby，Closure.）。
    
这篇文章的重点在于JVM规范中描述的运行时数据区（Runtime Data Areas）。这些区域被设计为存储程序或由JVM本身使用的数据。首先我会概述，然后什么是字节码，最后解释不同的数据区域。

<!--more-->
## 概述

JVM是对底层操作系统的抽象，不管底层的硬件和系统是否有差异，相同的代码在JVM上运行的结果保证一致。
example:
- 整型的大小永远都是`32`位，范围 `-2^31~2^31-1`。
- 存储数据使用大端模式（高地址存放低位）

![JVM](http://coding-geek.com/wp-content/uploads/2015/04/jvm_overview.jpg)

- Java代码先编译成字节码文件，JVM解析字节码文件。
- 为了避免磁盘I/O，JVM通过类加载器，报字节码文件加载到运行时数据区（RDA）。这些字节码会一直保存，直到JVM停止或类加载器销毁。
- 加载的字节码被执行引擎解析并执行。
- 执行引擎会保存正在执行的状态，以及与底层操作系统的协作。

**注意**：经常使用的代码会被编译成本地代码。这叫做即时编译（JIT），加快运行速度。

## 基于堆栈的架构

JVM使用基于堆栈的架构。虽然，对于开发者它是不可见的，但是它对生成的字节码和JVM架构有巨大的影响。

JVM通过执行的Java字节码中描述的基本操作来执行开发人员的代码。操作数是指令操作的值。根据JVM规范，这些操作需要在操作栈中进行。
![stack](http://coding-geek.com/wp-content/uploads/2015/04/state_of_java_operand_stack.jpg)

* 将操作数3，4入栈
* 调用加法指令
* 3，4出栈
* 计算3+4，将结果7入栈

**注意**： 其它的架构，如基于**寄存器**的架构，`X86架构`的处理器和Android虚拟机`Dalvik`使用。

## 字节码

Java一个字节组成的操作码，有256中，其中，`204`个是Java8使用规范的。
比如：
- `ifeq(0x99)` 表示equals
- `iadd(0x66)` 表示add
- `i2l(0x85) `integer 转 long
- `arraylength(0xbe)` 给出数组长度
- `pop（0x57）`堆栈第一个元素出栈

简答的加法
```java
public class Test {
  public static void main(String[] args) {
    int a =1;
    int b = 15;
    int result = add(a,b);
  }
 
  public static int add(int a, int b){
    int result = a + b;
    return result;
  }
}
```
JDK提供javap，可以将字节码转成可以理解的语句。
`javap -verbose Test.class`
```java
Classfile /C:/TMP/Test.class
  Last modified 1 avr. 2015; size 367 bytes
  MD5 checksum adb9ff75f12fc6ce1cdde22a9c4c7426
  Compiled from "Test.java"
public class com.codinggeek.jvm.Test
  SourceFile: "Test.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#15         //  java/lang/Object."<init>":()V
   #2 = Methodref          #3.#16         //  com/codinggeek/jvm/Test.add:(II)I
   #3 = Class              #17            //  com/codinggeek/jvm/Test
   #4 = Class              #18            //  java/lang/Object
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = Utf8               Code
   #8 = Utf8               LineNumberTable
   #9 = Utf8               main
  #10 = Utf8               ([Ljava/lang/String;)V
  #11 = Utf8               add
  #12 = Utf8               (II)I
  #13 = Utf8               SourceFile
  #14 = Utf8               Test.java
  #15 = NameAndType        #5:#6          //  "<init>":()V
  #16 = NameAndType        #11:#12        //  add:(II)I
  #17 = Utf8               com/codinggeek/jvm/Test
  #18 = Utf8               java/lang/Object
{
  public com.codinggeek.jvm.Test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
 
  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
         0: iconst_1
         1: istore_1
         2: bipush        15
         4: istore_2
         5: iload_1
         6: iload_2
         7: invokestatic  #2                  // Method add:(II)I
        10: istore_3
        11: return
      LineNumberTable:
        line 6: 0
        line 7: 2
        line 8: 5
        line 9: 11
 
  public static int add(int, int);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=2
         0: iload_0
         1: iload_1
         2: iadd
         3: istore_2
         4: iload_2
         5: ireturn
      LineNumberTable:
        line 12: 0
        line 13: 4
}
```
从上可知，不只是简单的转录Java类
- 包含了`常量池`的描述。常量池JVM数据区的一部分，用于存储类的元数据，如`方法名称、参数`...。当一个类被加载到JVM中，这些部分就会放在常量池中。
- `LineNumberTable` 和 `LocalVariableTable` 指定了方法的位置和方法的变量。
- 加上了`默认构造函数`
- 指定了堆栈的操作信息。


## 运行时数据区

运行时数据区是设计用来存储数据的内存区域。这些数据是由开发者的程序或JVM其内部工作使用。

![runtime-data-ares](http://coding-geek.com/wp-content/uploads/2015/04/jvm_memory_overview.jpg)

**堆**

堆是所有JVM线程之间共享的内存区域。它是在虚拟机启动时创建的。所有的实例对象和数组被分配到堆中。
如：
```java
MyClass myVariable = new MyClass();
MyClass[] myArrayClass = new MyClass[1024];
```
堆必须要由垃圾回收器（`GC`）管理，当对象不使用时，由GC回收被分配的内存。GC的回收策略由取决于具体的JVM实现(Oracle Hotspot 就实现了很多算法）

堆可以动态扩大或收缩，也可以有固定的最大值和最小值。比如，在Oracle Hotspot里，用户可以指定参数：`java -Xms=512m -Xmx=1024m`

**注意**：堆不能超过最大值，否则会抛出`OutOfMemoryError`

**方法区**

方法区也是所有JVM线程共享的内存区域。也是在虚拟机启动时创建，并通过类加载器加载。方法区会一直保持，知道类加载器关闭。
方法区保存：
> 
- 类信息（字段、方法、父类名称、接口名称、版本...）
- 方法和构造函数的字节码表示
- 每个类都要加载的运行时常量池

JVM规范并没有强制要求方法区要在堆中实现。例如，Oracle HotSpot使用一种叫`PermGen`的区域来存储方法区。这个PermGen区邻居Java的堆，并限制了默认空间大小为64M（`XX：MaxPermSize` 可以修改）。从java 8开始，HotSpot把方法区存放在叫做`Metaspace`的独立的本地内存中。最大的可用空间等于系统可用的内存空间。

**注意**：如果方法区超过最大的范围，就会抛出`OutOfMemoryError`

**运行时常量池**

**运行时常量池是方法区的子集**。因为它是元数据的重要组成部分，Oracle规定将运行时常量池从方法区中分离出来。这个常量池会随着加载的类和接口而增加。这个常量池就像传统编程语言中的**符号表**。换句话说，当一个类、方法或字段被引用时，JVM会使用运行时常量池查找**真实的内存地址**。同时，常量池也保存**字符串**和**基本类型**的**常量值**。
```java
String myString1 = “This is a string litteral”;
static final int MY_CONSTANT=2;
```

**PC寄存器（每线程）**

每个线程都有自己的`PC（程序计数器）寄存器`，与线程同时创建。任何时候，每一个线程都在执行当前的方法。寄存器指向当前正在执行的命令（在方法区）。

**注意**：如果方法被本地线程执行，那么寄存器的值是未知的。JVM的寄存器足够保存返回地址或指针。

**Java虚拟机栈（每个线程）**

Java栈中保存了很多帧。

**帧**

帧是一种数据结构，它包含了很多数据，这些数据表示正在执行当前方法的线程状态。

- 操作数栈
- 本地变量数组
- 运行时常量池引用

**栈**

每个Java虚拟机线程有一个专用的Java虚拟机栈，与线程同时创建。栈保存了帧。每调一次方法，新的帧就会被放到栈中。当方法调用结束，帧也销毁了。不管是否正常结束。
只有执行方法的帧，总是保持活动的。这个就是当前帧，它涉及当前方法和当前类。

example
```java
public int add(int a, int b){
  return a + b;
}
 
public void functionA(){
// some code without function call
  int result = add(2,3); //call to function B
// some code without function call
}
```
![stack](http://coding-geek.com/wp-content/uploads/2015/04/state_of_jvm_method_stack.jpg)

注意：栈是动态的，但是有一个最大尺寸限制，如果递归调用过多，或引发`StackOverflowError`
Oracle HotSpot 可以修改`-Xss`

**本地方法栈（每个线程）**

只是为非Java语言写的本地方法（JNI）使用的栈，所以有操作系统控制。

## 总结

希望这篇文章可以帮你更好的理解JVM。我觉得栈是重点，因为涉及到JVM的内部功能。
如果想要深入理解：

- 你可以看看[JVM规范](http://docs.oracle.com/javase/specs/jvms/se8/html/index.html)
- 推荐[Understanding JVM Internals](http://www.cubrid.org/blog/dev-platform/understanding-jvm-internals/)