---
title: Java 8 避免空检查
tags: [Java]
categories: [Java]
photo:
  - /images/java8.png
date: 2017-04-19 22:16:29
---
# Java 8 避免空检查
---

## 概述
`null`的发明者`Tony Hoare`在2009年致歉，表述了这个**billion-dollar mistake**。
> 
I call it my billion-dollar mistake. It was the invention of the null reference in 1965. At that time, I was designing the first comprehensive type system for references in an object oriented language (ALGOL W). My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. **But I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement.** This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.

<!--more-->
## 举例

- 定义三层嵌套的类
```java
class Outer {
    Nested nested;
    Nested getNested() {
        return nested;
    }
}
class Nested {
    Inner inner;
    Inner getInner() {
        return inner;
    }
}
class Inner {
    String foo;
    String getFoo() {
        return foo;
    }
}
```
- 原始避免`NullPointerException`的方法很繁琐
```java
Outer outer = new Outer();
if (outer != null && outer.nested != null && outer.nested.inner != null) {
    System.out.println(outer.nested.inner.foo);
}
```
- Java 8 使用`Optinal`的`map`，它接受一个`Function`，封装为`Optional`对象。
```java
Optional.of(new Outer())
    .map(Outer::getNested)
    .map(Nested::getInner)
    .map(Inner::getFoo)
    .ifPresent(System.out::println);
```
- 另一种通过`Supplier`方法 (个人不推荐主动捕获`NullPointerException`）
```java
public static <T> Optional<T> resolve(Supplier<T> resolver) {
    try {
        T result = resolver.get();
        return Optional.ofNullable(result);
    }
    catch (NullPointerException e) {
        return Optional.empty();
    }
}
```
```java
Outer obj = new Outer();
resolve(() -> obj.getNested().getInner().getFoo());
    .ifPresent(System.out::println);
```
调用`obj.getNested().getInner().getFoo()`可能抛出`NullPointerException`。
这里手动捕获了异常，并返回O`ptional.empty()`
