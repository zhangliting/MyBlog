---
title: Spring 配置文件最佳实践
tags: [Spring,实践]
categories: [Spring]
photo:
  - /images/pose.jpg
date: 2017-05-16 21:55:55
---
# Spring 配置文件最佳实践

## 1. 给配置文件加上注释
```xml
<beans>
    <description>
        This configuration file will have all beans 
        which may be used for controlling transactions.
    </description>
    ...
</beans>
```
## 2. 使用统一的命名规范
比如：EmployeeUpdateDAO的ID命名为employeeUpdateDAO

<!--more-->
## 3. 引用schema不使用版本号
Spring会自动从项目依赖中选择最高的版本，随着Spring版本升级，我们不需要去维护配置文件schema。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
  
  <!-- http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
  http://www.springframework.org/schema/context 
  http://www.springframework.org/schema/context/spring-context-3.0.xsd">-->
  
  <!--  replace -->
  
  http://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/context 
  http://www.springframework.org/schema/context/spring-context.xsd">
   
  <!-- Other bean definitions-->
   
</beans>
```
## 4. Setter注入优先于Constructor
Setter注入更加灵活和好维护。
## 5. 使用bean引用
推荐使用shorter version 
```xml
<!-- Expanded version -->
<bean id="employeeDAO" class="com.howtodoinjava.dao.EmployeeDAO">
    <property name="datasource">
        <ref bean="datasource"></ref>
        <value>datasource</value>
     </property>
</bean>
 
<!-- Shorter/shortcut version -->
<bean id="employeeDAO" class="com.howtodoinjava.dao.EmployeeDAO">
    <property name="datasource"  ref="datasource" value="datasource">
</bean>
```
## 6. 重用bean定义
```xml
<bean id="abstractDataSource" class="org.apache.commons.dbcp.BasicDataSource"
    destroy-method="close"
    p:driverClassName="${jdbc.driverClassName}"
    p:username="${jdbc.username}"
    p:password="${jdbc.password}" />
 
<bean id="concreteDataSourceOne"
    parent="abstractDataSource"
    p:url="${jdbc.databaseurlOne}"/>
  
<bean id="concreteDataSourceTwo"
    parent="abstractDataSource"
    p:url="${jdbc.databaseurlTwo}"/>
```
## 7. 总是使用id作为bean的标示
## 8. 总是使用classpath作为前缀
当使用资源，配置文件，属性文件时，总是使用classpath作为前缀。
通常`classpath`位于`src/main/java` `src/main/resources` `src/test/java` `src/test/resources`
```xml
<!-- Always use classpath: prefix-->
<import resource="classpath:/META-INF/spring/applicationContext-security.xml"/>
```
## 9. 引用外部属性文件

`jdbc.properties`
```
/* file://jdbc.properties */
 
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.username=root
jdbc.password=password
```
```xml
<bean id="abstractDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="${jdbc.driverClassName}"
        p:username="${jdbc.username}"
        p:password="${jdbc.password}" />
```
## 10. 不要过度使用依赖注入
如：DTO