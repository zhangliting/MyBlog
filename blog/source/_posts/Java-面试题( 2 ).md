---
title: Java 面试题(2)
tags: [Java 面试]
categories: [Java]
photo:
  - /images/java.png
date: 2017-04-18 22:34:51
---
# Java 面试题（2）
---
## 避免使用finalize()方法
finalize()方法会被GC线程调用，但不保证执行。 
> 
* finalize（）方法不会像constructor()一样调用super finalize()。
* finalize()方法抛出的异常会被GC忽略。
* finalize()降低性能。

## notify()和wait()为什么定义在Object class
> 
* notify(), wait(), notifyAll()，用于多线程共享资源。
* 定义在Object class，所有的对象都可以控制调用它的线程必须等待monitor。
* Java使用Hoare's monitors idea。
* Java不能指定某个线程运行，总是当前的线程远行代码。但是可以指定监视器（调用wait()等待获取监视器,notify()释放监视器））。
* 这是一个好的设计，如果一个线程可以随时改变其他线程等待监视器，就会导致`侵入`。这是不提倡的，就像stop()。

<!--more-->

## DeadLock Example

死锁:两个线程同时持有一些不同的资源，同时请求对方的资源。

一个死锁的例子：
```java
package thread;
 
public class ResolveDeadLockTest {
 
    public static void main(String[] args) {
        ResolveDeadLockTest test = new ResolveDeadLockTest();
        //创建2个资源
        final A a = test.new A();
        final B b = test.new B();
 
        // Thread-1
        Runnable block1 = new Runnable() {
            public void run() {
                synchronized (a) {
                    try {
                        // Adding delay so that both threads can start trying to
                        // lock resources
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // Thread-1 have A but need B also
                    synchronized (b) {
                        System.out.println("In block 1");
                    }
                }
            }
        };
 
        // Thread-2
        Runnable block2 = new Runnable() {
            public void run() {
                synchronized (b) {
                    // Thread-2 have B but need A also
                    synchronized (a) {
                        System.out.println("In block 2");
                    }
                }
            }
        };
 
        new Thread(block1).start();
        new Thread(block2).start();
    }
 
    // Resource A
    private class A {
        private int i = 10;
 
        public int getI() {
            return i;
        }
 
        public void setI(int i) {
            this.i = i;
        }
    }
 
    // Resource B
    private class B {
        private int i = 20;
 
        public int getI() {
            return i;
        }
 
        public void setI(int i) {
            this.i = i;
        }
    }
}
```
解决方法：`多个线程调用资源的顺序一致`，a->b。

## Transient

>Transient关键字表示一个字段不应该被序列化和持久化。

## Volatile

- 每一个线程都有自己的本地内存空间，线程的读写操作都在本地内存空间上进行。当所有操作完成，线程把本地内存中修改的变量写回到主内存空间。
- volatile告诉JVM，保证本地内存空间的变量和主内存一致。每一次线程读取某个volatiel修饰的变量时，都会去主内存获取最新的值。
- volatile也可以防止指令重排。

## Iterator and ListIterator 区别

- Iterator：Set，List，Map。
- ListIterator：List。
- ListIterator can
    - iterate backwards.
    - obtain the index at any point.
    - add a new value at any point.
    - set a new value at that point.
    
## Copy

* Java 默认是shallow copy, 基本类型会创建相同的对象。引用类只会创建同一个引用。
* Java实现copy
    1. 实现Cloneable接口。
    2. 覆写clone()方法。
* Example
```java
public class Employee implements Cloneable{
 
    private int empoyeeId;
    private String employeeName;
    private Department department;// shallow copy
 
    public Employee(int id, String name, Department dept)
    {
        this.empoyeeId = id;
        this.employeeName = name;
        this.department = dept;
    }
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
     
    //Accessor/mutators methods will go there
}
```
* 手动修改Department to deep copy
```java
/Modified clone() method in Employee class
@Override
protected Object clone() throws CloneNotSupportedException {
    Employee cloned = (Employee)super.clone();
    cloned.setDepartment((Department)cloned.getDepartment().clone());
    return cloned;
}
```
同时Departments要实现clone()。
```java
//Defined clone method in Department class.
@Override
protected Object clone() throws CloneNotSupportedException {
    return super.clone();
}
```
* 序列化实现深拷贝
```java
public SerializableClass deepCopy() throws Exception
    {
        //Serialization of object
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream out = new ObjectOutputStream(bos);
        out.writeObject(this);
 
        //De-serialization of object
        ByteArrayInputStream bis = new   ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream in = new ObjectInputStream(bis);
        SerializableClass copied = (SerializableClass) in.readObject();
 
        //Verify that object is not corrupt
 
        //validateNameParts(fName);
        //validateNameParts(lName);
 
        return copied;
    }
```

* Using Apache commons
  内部使用序列化，deep copy
```java
SomeObject cloned = org.apache.commons.lang.SerializationUtils.clone(someObject);
```
* Best Practices
```java
if(obj1 instanceof Cloneable){
    obj2 = obj1.clone();
}
 
//Dont do this. Cloneabe dont have any methods
obj2 = (Cloneable)obj1.clone()
```

## && and & 区别

* & 是位运算符，&&是逻辑运算符。
* && 左边是true，就不会执行右边的。

## 访问修饰符
|Modifiers|	Same Class|	Same Package|	Subclass|	Other packages|
|---------|--|--|---|----|
|public	  | Y|	Y|	Y|	Y|
|protected|	Y|	Y|	Y|	N|
|default|	Y|	Y|	N|	N|
|private|	Y|	N|	N|	N|


## Java native 方法
> native关键字修饰的方法，使用JNI调用非Java的方法。
1. 调用非Java写的方法。
2. 需要调用系统资源相关的方法