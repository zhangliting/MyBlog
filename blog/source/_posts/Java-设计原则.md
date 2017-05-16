---
title: Java 设计原则
tags: [Java]
categories: [Java]
photo:
  - /images/solid_class_design_principles.png
date: 2017-04-19 22:26:09
---
# Java 设计原则

5个Java类设计原则

![java-design](/images/solid_class_design_principles.png)

## 单一职责原则
>一个类有且只有一个职责。

* 一个entity对应一张表。
* service类不应该含有工具方法。
* 便于维护。

<!--more-->
## 开闭原则

> 软件组件应该对扩展开放，对修改关闭。

* 像Spring框架，类的核心功能不能被修改，但你可以覆写某些方法，以满足你的需求。

## 里氏的替换原则

> 派生的类型必须完全可以替代它们的父类。

* 子类可以实现父类的抽象方法，但不能改变父类原有的方法。
* 子类可以增加自己的方法


## 接口隔离原则

> 客户不应该被强迫实现他们不必要的方法。

* 如果借口如果借口`eportable`有两个方`法generateExcel()`，`eneratedPdf()`。但是我只需要生成PDF，这就需要把两个方法分别放在`PdfReportable`，`ExcelReportable`。


## 依赖倒置原则

>依赖抽象，而不是具体实现。

* Spring framework的BeanFactory就是经典的例子。
* 所有的模块都是分离的，需要的模块通过依赖注入。
* 降低模块的耦合，便于维护和扩展。