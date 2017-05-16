---
title: Java面试题（1）
tags: [Java 面试]
categories: [Java]
photo:
  - /images/java.png
date: 2017-04-18 01:22:48
---
# Java 面试题(1)
---

## 创建不可修改的对象
> 
1. 不提供`setter`方法
2. 实例变量声明`private final`
3. `final class` 修饰类，防止继承
4. 方法返回时，final修饰的实例变量可以直接返回，否则返回copy。

**不可变对象的优点**

* 方便创建、使用和测试。
* 线程安全的（String）
* 适合作为`Map`和`Set`的Key。
* 允许`hashcode`延迟初始化和缓存
 
 <!--more-->
**example**
```java
import java.util.Date;
 
/**
* Always remember that your instance variables will be either mutable or immutable.
* Identify them and return new objects with copied content for all mutable objects.
* Immutable variables can be returned safely without extra effort.
* */
public final class ImmutableClass
{
 
    /**
    * Integer class is immutable as it does not provide any setter to change its content
    * */
    private final Integer immutableField1;
    /**
    * String class is immutable as it also does not provide setter to change its content
    * */
    private final String immutableField2;
    /**
    * Date class is mutable as it provide setters to change various date/time parts
    * */
    private final Date mutableField;
 
    //Default private constructor will ensure no unplanned construction of class
    private ImmutableClass(Integer fld1, String fld2, Date date)
    {
        this.immutableField1 = fld1;
        this.immutableField2 = fld2;
        this.mutableField = new Date(date.getTime());
    }
 
    //Factory method to store object creation logic in single place
    public static ImmutableClass createNewInstance(Integer fld1, String fld2, Date date)
    {
        return new ImmutableClass(fld1, fld2, date);
    }
 
    //Provide no setter methods
 
    /**
    * Integer class is immutable so we can return the instance variable as it is
    * */
    public Integer getImmutableField1() {
        return immutableField1;
    }
 
    /**
    * String class is also immutable so we can return the instance variable as it is
    * */
    public String getImmutableField2() {
        return immutableField2;
    }
 
    /**
    * Date class is mutable so we need a little care here.
    * We should not return the reference of original instance variable.
    * Instead a new Date object, with content copied to it, should be returned.
    * */
    public Date getMutableField() {
        return new Date(mutableField.getTime());
    }
 
    @Override
    public String toString() {
        return immutableField1 +" - "+ immutableField2 +" - "+ mutableField;
    }
}
```
## Java是引用传递?值传递？
> everything in java is pass-by-value

example
```java
public class Foo
{
    private String attribute;
 
    public Foo (String a){
        this.attribute = a;
    }
    public String getAttribute() {
        return attribute;
    }
    public void setAttribute(String attribute) {
        this.attribute = attribute;
    }
}
 
public class Main
{
     public static void main(String[] args){
          Foo f = new Foo("f");
          changeReference(f); // It won't change the reference!
          modifyReference(f); // It will change the object that the reference variable "f" refers to!
     }
     public static void changeReference(Foo a) {
          Foo b = new Foo("b");
          a = b;
     }
     public static void modifyReference(Foo c) {
          c.setAttribute("c");
     }
}
```
在函数中，虚参是实参的拷贝，它们指向同一个对象，改变虚参的引用，并不会改变实参的引用，但是改变虚参指向的对象的属性，会同时改变实参指向的对象的属性。

## finally 关键字
* finally总是在try后执行。
* finally不只用于处理异常，更适合处理资源的回收和代码清理工作。
* JVM异常退出时，finally不一定会执行。

## java.util.Date VS java.sql.Date
* java.util.Date == java.sql.Date + java.sql.Time
* java.sql.Date extend java.util.Date

## 标签接口（marker interface）
> It provices means to associate metadata with class where the languages does not hava explicit support for such metadata.
一个类实现了标签接口，代表具有某种能力。

example 
`Serializable` `Cloneable`

## main方法为什么声明为public static void

**public**
需要任何地方，任何对象都可以访问，因为它是程序的入口。
**static** 
如果不是static，就需要创建对象，而且main传递可变参数，就会存在参数重载，就不知道那个main方法。
**void**
JVM调用main方法不需要接受返回参数。
可以使用`System.exit(int)`告诉JVM是否正常退出。0-正常。

## 创建String：new() 和 字面量的区别
* new() 会同时在JVM heap和字符串常量池（JVM heap的永久区）里创建，而字面量只在后者创建。

**example**
```java
String str = "abc";//if string pool exist "abc", reuse this, or add to string pool.
String str = new String("abc");//create "abc" in string pool if not exist, and create obj in heap too.(two objects created)
```
```java
String str = new String("abc");
// force use string pool
str.intern();
```
### 为什么String是不可变的
* 提高性能（string pool, reuse）
* 安全考虑（thread security, memory leak issues）

### String比较
* == 比较对象引用，同一个对象或者同一个strig pool的引用，返回true。
* equals 被String覆写，比较字符的ascii值。

### Menory leak issue
String对象使用substring(int beginIndex) 创建对象的时候，value[]不会变，只改变了字符串开始和结束的索引。引起内存的浪费。
**substring**
```java
/** The value is used for character storage. */
private final char value[];

To access this array in different scenarios, following variables are used:

/** The offset is the first index of the storage that is used. */
private final int offset;

/** The count is the number of characters in the String. */
private final int count;
```
**example**
```java
import java.lang.reflect.Field;
import java.util.Arrays;
 
public class SubStringTest {
    public static void main(String[] args) throws Exception
    {
        //Our main String
        String mainString = "i_love_java";
        //Substring holds value 'java'
        String subString = mainString.substring(7);
 
        System.out.println(mainString);
        System.out.println(subString);
 
        //Lets see what's inside mainString
        Field innerCharArray = String.class.getDeclaredField("value");
        innerCharArray.setAccessible(true);
        char[] chars = (char[]) innerCharArray.get(mainString);
        System.out.println(Arrays.toString(chars));
 
        //Now peek inside subString
        chars = (char[]) innerCharArray.get(subString);
        System.out.println(Arrays.toString(chars));
    }
}
```
**解决方法**
```java
//Our new method prevents memory leakage
public static String fancySubstring(int beginIndex, String original)
{
    return new String(original.substring(beginIndex));
}   
```

## interface 和 abstract class 区别

- interface中定义的变量都是public final，abstract class可以包含非final得变量。
- interface中定义的方法都是public abstract抽象方法，abstract class可以有实现的方法。
- interface using implements, abstract class using exntends。
- Java类可以实现多个接口，但是只能继承一个父类（单继承）。
- 都不可以实例化。

## 什么时候覆写hashCode()和equals()方法
hashCode()和equals()方法都是定义在Object类中。
```java
// return integer representation of memory address where object is stored
public native int hashCode();
// check reference of two objects
public boolean equals(Object obj) {
    return (this == obj);
}
```
* `equals()`必须是自反的，对称的和传递的。o.equals(null)始终返回false。
* `equals()相等，那么hashCode()也保持相等。