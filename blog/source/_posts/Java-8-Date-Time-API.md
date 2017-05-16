---
title: Java 8 Date Time API
tags: [Java]
categories: [Java]
photo:
  - /images/java-time.jpg
date: 2017-04-24 20:54:16
---
# Java 8 Date Time API
---
Clock为instant,date,time 提供时间，是包含时区的。
```java
// get the current time
Clock clock = Clock.systemDefaultZone();
long t0 = clock.millis();
//Clock中获得Instant,载将Instant转Date
Instant instant = clock.instant();
Date legacyDate = Date.from(instant);
```

<!--more-->
## java.time.Instant

Instant表示时间线上的一个点。

```java
Instant now = Instant.now();//获得当前时间 不含时区 UTC时间
//2014-09-20T14:32:33.646Z 

//从1970-01-01T00:00:00Z开始的毫秒数
System.out.println(now.getEpochSecond());
// prints 1411137153

//加减
Instant tomorrow = now.plus(1,ChronoUnit.DAYS);
Instant tomorrow = now.plus(1,ChronoUnit.DAYS);

//比较

System.out.println(now.compareTo(tomorrow));// prints -1

System.out.println(now.isAfter(yesterday));// prints true
```

## java.time.LocalDate

LocalDate保存日期部分，不包含时区，是不可变得类。
```java
//当前日期
LocalDate today = LocalDate.now();
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
LocalDate yesterday = tomorrow.minusDays(2);
//自定义日期
LocalDate independenceDay = LocalDate.of(2014, Month.JULY, 4);
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
System.out.println(dayOfWeek);    // FRIDAY
//格式化日期
DateTimeFormatter germanFormatter =
        DateTimeFormatter
                .ofLocalizedDate(FormatStyle.MEDIUM)
                .withLocale(Locale.GERMAN);

LocalDate xmas = LocalDate.parse("24.12.2014", germanFormatter);
System.out.println(xmas);   // 2014-12-24
```
## java.time.LocalTime

LocalTime保存时间部分，不包含时区，是不可变得类。
```java
//当前时间
LocalTime now = LocalTime.now();
//自定义时间
LocalTime late = LocalTime.of(23, 59, 59);
System.out.println(late);
//格式化时间
DateTimeFormatter germanFormatter =
        DateTimeFormatter
                .ofLocalizedTime(FormatStyle.SHORT)
                .withLocale(Locale.GERMAN);

LocalTime leetTime = LocalTime.parse("13:37", germanFormatter);
System.out.println(leetTime);
```
## java.time.LocalDateTime

LocalDateTime同时包括了时间和日期。不包含时区，是不可变得类。
```java
//格式化
DateTimeFormatter formatter =
        DateTimeFormatter
                .ofPattern("MMM dd, yyyy - HH:mm");

LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);
String string = parsed.format(formatter);
System.out.println(string);     // Nov 03, 2014 - 07:13
```

        


