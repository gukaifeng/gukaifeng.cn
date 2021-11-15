---
title: Java中的时间类
mathjax: false
date: 2020-03-20 13:18:57
updated: 2020-03-20 13:18:57
tags: [Java,JavaSE]
categories: [编程语言概念]
toc: true
---

Java中有多种时间相关的类。

* java.util.Date（基本废弃，Deprecated）

    java.util.Date中的getTime()方法还算常用，返回自1970-01-01 00:00:00以来的毫秒数。

    ```java
    Date d = new Date();
    System.out.println(d);
    System.out.println(d.getTime()); 
    //the number of milliseconds since 1970.1.1 00:00:00 
    ```

    ```
    Fri Mar 20 15:08:36 CST 2020
    1584688116674
    ```

* java.sql.Date（和数据库对应使用的时间类）

* **java.util.Calendar**是目前程序中最常使用的，但是是抽象类，使用了简单的工厂模式。

    Calendar.getInstance();返回的是GregorianCalendar对象

    ```java
    import java.util.Calendar;
    import java.util.GregorianCalendar;
    
    public class CalendarClassTest {
    
        public static void main(String[] args) {
            Calendar gc = Calendar.getInstance();
            System.out.println(gc.getClass().getName());
            //Calendar.getInstance();返回的是GregorianCalendar对象
    
            GregorianCalendar gc2 = new GregorianCalendar();
            System.out.println(gc2.getClass().getName());
        }
    
    }
    ```

    ```
    java.util.GregorianCalendar
    java.util.GregorianCalendar
    ```

* **java.time**

<!--more-->



## 1. java.util.Calendar

Calender类中的主要函数方法有

* get(Field) 获取时间中每个属性的值。注意月份是0-11，使用时要+1。
* getTime() 返回相应的java.utim.Date对象。
* getTimeInMillis 返回自1970-01-01 00:00:00以来的毫秒数。
* set(Field) 设置时间字段。
* add(field, amount) 将指定字段增加/减少时间
* roll(field, amount) 将指定字段增加/减少时间，但不影响上一级的时间段（即不进位/退位）。

下面通过代码来展示上面几种方法，相关解释写在了注释里。

```java
import java.util.Calendar;

public class CalendarTest {

    Calendar calendar = Calendar.getInstance();

    public void test1() {
        // 获取年
        int year = calendar.get(Calendar.YEAR);
        // 获取月，这里需要需要月份的范围为0~11，因此获取月份的时候需要+1才是当前月份值
        int month = calendar.get(Calendar.MONTH) + 1;
        // 获取日
        int day = calendar.get(Calendar.DAY_OF_MONTH);

        // 获取时
        int hour = calendar.get(Calendar.HOUR);
        // int hour = calendar.get(Calendar.HOUR_OF_DAY); // 24小时表示
        // 获取分
        int minute = calendar.get(Calendar.MINUTE);
        // 获取秒
        int second = calendar.get(Calendar.SECOND);

        // 星期，英语国家星期从星期日开始计算
        int weekday = calendar.get(Calendar.DAY_OF_WEEK);

        System.out.println("现在是" + year + "年" + month + "月" + day + "日" + hour
                + "时" + minute + "分" + second + "秒" + "星期" + weekday);
    }

    // 一年后的今天
    public void test2() {
        // 同理换成下个月的今天calendar.add(Calendar.MONTH, 1);
        calendar.add(Calendar.YEAR, 1);

        // 获取年
        int year = calendar.get(Calendar.YEAR);
        // 获取月
        int month = calendar.get(Calendar.MONTH) + 1;
        // 获取日
        int day = calendar.get(Calendar.DAY_OF_MONTH);

        System.out.println("一年后的今天：" + year + "年" + month + "月" + day + "日");
    }

    // 获取任意一个月的最后一天
    public void test3() {
        // 假设求6月的最后一天
        // 先求出7月份的第一天，实际中这里Calendar.JULY = 6
        calendar.set(calendar.get(Calendar.YEAR), Calendar.JULY, 1);

        calendar.add(Calendar.DATE, -1);

        // 获取日
        int day = calendar.get(Calendar.DAY_OF_MONTH);

        System.out.println("6月份的最后一天为" + day + "号");
    }

    // 设置日期
    public void test4() {
        calendar.set(Calendar.YEAR, 2000);
        System.out.println("现在是" + calendar.get(Calendar.YEAR) + "年");

        calendar.set(2018, 7, 8);
        // 获取年
        int year = calendar.get(Calendar.YEAR);
        // 获取月
        int month = calendar.get(Calendar.MONTH)+1;
        // 获取日
        int day = calendar.get(Calendar.DAY_OF_MONTH);

        System.out.println("现在是" + year + "年" + month + "月" + day + "日");
    }

    //add和roll的区别
    public void test5() {

        calendar.set(2018, 7, 8);
        calendar.add(Calendar.DAY_OF_MONTH, -8);

        // 获取年
        int year = calendar.get(Calendar.YEAR);
        // 获取月
        int month = calendar.get(Calendar.MONTH)+1;
        // 获取日
        int day = calendar.get(Calendar.DAY_OF_MONTH);

        System.out.println("2018.8.8, 用add减少8天，现在是" + year + "." + month + "." + day);

        calendar.set(2018, 7, 8);
        calendar.roll(Calendar.DAY_OF_MONTH, -8);

        // 获取年
        year = calendar.get(Calendar.YEAR);
        // 获取月
        month = calendar.get(Calendar.MONTH)+1;
        // 获取日
        day = calendar.get(Calendar.DAY_OF_MONTH);

        System.out.println("2018.8.8, 用roll减少8天，现在是" + year + "." + month + "." + day);
    }


    public static void main(String[] args) {
        CalendarTest c = new CalendarTest();
        c.test1();
        System.out.println("============");
        c.test2();
        System.out.println("============");
        c.test3();
        System.out.println("============");
        c.test4();
        System.out.println("============");
        c.test5();

    }

}

```

```
现在是2020年3月20日1时33分22秒星期6
============
一年后的今天：2021年3月20日
============
6月份的最后一天为30号
============
现在是2000年
现在是2018年8月8日
============
2018.8.8, 用add减少8天，现在是2018.7.31
2018.8.8, 用roll减少8天，现在是2018.8.31
```



## 2. java.time

旧的时间类设计不好，如重名的类（Java.util.Data和Java.sql.Data），线程不安全（Calendar）等。

java.time是Java 8推出的新的时间API。

* 不变性，适合在多线程下使用。
* 遵循设计模式，设计得更好，可扩展性强。

Java 8时间包可以分为五个类别，java.time包和其4个子包。

* **java.time包：新的Java日期/时间API基础包。** 
* java.time.chrono包：为非ISO的日历系统定义了一些泛化的API。
* java.time.format包：格式化和解析日期时间对象的类。
* java.time.temporal包：包含一些时态对象，可以用其找出日期/时间对象的某个特定的日期或时间。
* java.time.zone包：包含支持不同时区以及相关规则的类。



java.time包主要类：

* LocalDate：日期类，只负责年月日。
* LocalTime：时间类，只负责时分秒纳秒。
* LocalDateTime：LocalDate + LocalTime。
* Instant：时间戳，可以用来和java8以前的时间类互通。



下面通过代码示例来了解java.time包中的几个主要类，相关说明写在注释中。

### 2.1. LocalDate

```java
import java.time.LocalDate;
import java.time.Month;
import java.time.ZoneId;

public class LocalDateExample {

    public static void main(String[] args) {

        //当前时间
        LocalDate today = LocalDate.now();
        System.out.println("Current Date="+today);

        //根据指定时间创建LocalDate
        LocalDate firstDay_2014 = LocalDate.of(2014, Month.JANUARY, 1);
        System.out.println("Specific Date="+firstDay_2014);

        //给定错误时间参数，将报异常java.time.DateTimeException
        //LocalDate feb29_2014 = LocalDate.of(2014, Month.FEBRUARY, 29);

        //可以更改时区
        LocalDate todayBeijing = LocalDate.now(ZoneId.of("Asia/Shanghai"));
        System.out.println("Current Date in Shanghai="+todayBeijing);

        //从纪元日01/01/1970开始365天
        LocalDate dateFromBase = LocalDate.ofEpochDay(365);
        System.out.println("365th day from base date= "+dateFromBase);

        //2014年的第100天
        LocalDate hundredDay2014 = LocalDate.ofYearDay(2014, 100);
        System.out.println("100th day of 2014="+hundredDay2014);
    }

}
```

```
Current Date=2020-03-20
Specific Date=2014-01-01
Current Date in Shanghai=2020-03-20
365th day from base date= 1971-01-01
100th day of 2014=2014-04-10
```

### 2.2. LocalTime

```java
import java.time.LocalTime;
import java.time.ZoneId;

public class LocalTimeExample {

    public static void main(String[] args) {

        //当前时间  时分秒 纳秒
        LocalTime time = LocalTime.now();
        System.out.println("Current Time="+time);

        //根据时分秒
        LocalTime specificTime = LocalTime.of(12,20,25,40);
        System.out.println("Specific Time of Day="+specificTime);

        //错误的时间参数 将报DateTimeException
        //LocalTime invalidTime = LocalTime.of(25,20);

        //上海时间
        LocalTime timeSH = LocalTime.now(ZoneId.of("Asia/Shanghai"));
        System.out.println("Current Time in SH="+timeSH);


        //一天当中第几秒
        LocalTime specificSecondTime = LocalTime.ofSecondOfDay(10000);
        System.out.println("10000th second time= "+specificSecondTime);

    }

}
```

```
Current Time=14:51:37.999755
Specific Time of Day=12:20:25.000000040
Current Time in SH=14:51:38.002453
10000th second time= 02:46:40
```



### 2.3. LocalDateTime

```java
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.Month;
import java.time.ZoneId;
import java.time.ZoneOffset;

public class LocalDateTimeExample {

    public static void main(String[] args) {

        //当前日期 时分秒
        LocalDateTime today = LocalDateTime.now();
        System.out.println("Current DateTime="+today);

        //根据日期， 时分秒来创建对象
        today = LocalDateTime.of(LocalDate.now(), LocalTime.now());
        System.out.println("Current DateTime="+today);

        //指定具体时间来创建对象
        LocalDateTime specificDate = LocalDateTime.of(2014, Month.JANUARY, 1, 10, 10, 30);
        System.out.println("Specific Date="+specificDate);

        //如时间不对，将报异常DateTimeException
        //LocalDateTime feb29_2014 = LocalDateTime.of(2014, Month.FEBRUARY, 28, 25,1,1);

        //上海时区
        LocalDateTime todayShanghai = LocalDateTime.now(ZoneId.of("Asia/Shanghai"));
        System.out.println("Current Date in Shanghai="+todayShanghai);


        //从01/01/1970 10000秒
        LocalDateTime dateFromBase = LocalDateTime.ofEpochSecond(10000, 0, ZoneOffset.UTC);
        System.out.println("10000th second time from 01/01/1970= "+dateFromBase);
    }
}
```

```
Current DateTime=2020-03-20T14:53:18.624815
Current DateTime=2020-03-20T14:53:18.627766
Specific Date=2014-01-01T10:10:30
Current Date in Shanghai=2020-03-20T14:53:18.628215
10000th second time from 01/01/1970= 1970-01-01T02:46:40
```



### 2.4. Instant

```java
import java.time.Duration;
import java.time.Instant;
import java.util.Date;

public class InstantExample {

    public static void main(String[] args) {
        //当前时间戳
        Instant timestamp = Instant.now();
        System.out.println("Current Timestamp = "+timestamp);

        //从毫秒数来创建时间戳
        Instant specificTime = Instant.ofEpochMilli(timestamp.toEpochMilli());
        System.out.println("Specific Time = "+specificTime);

        Date date = Date.from(timestamp);
        System.out.println("current date = " + date);
    }

}
```

```
Current Timestamp = 2020-03-20T06:54:46.172833Z
Specific Time = 2020-03-20T06:54:46.172Z
current date = Fri Mar 20 14:54:46 CST 2020
```



### 2.5. java.time包的一些操作

```java
import java.time.LocalDate;
import java.time.LocalTime;
import java.time.Period;
import java.time.temporal.TemporalAdjusters;

public class DateUtil {

    public static void main(String[] args) {

        LocalDate today = LocalDate.now();

        //判断是否是闰年
        System.out.println("Year "+today.getYear()+" is Leap Year  "+today.isLeapYear());

        //今天和01/01/2015比较
        System.out.println("Today is before 01/01/2015  "+today.isBefore(LocalDate.of(2015,1,1)));

        //当前时分秒
        System.out.println("Current Time="+today.atTime(LocalTime.now()));

        //加减时间
        System.out.println("10 days after today will be "+today.plusDays(10));
        System.out.println("3 weeks after today will be "+today.plusWeeks(3));
        System.out.println("20 months after today will be "+today.plusMonths(20));

        System.out.println("10 days before today will be "+today.minusDays(10));
        System.out.println("3 weeks before today will be "+today.minusWeeks(3));
        System.out.println("20 months before today will be "+today.minusMonths(20));

        //调整时间
        System.out.println("First date of this month= "+today.with(TemporalAdjusters.firstDayOfMonth()));
        LocalDate lastDayOfYear = today.with(TemporalAdjusters.lastDayOfYear());
        System.out.println("Last date of this year= "+lastDayOfYear);

        //时间段计算
        Period period = today.until(lastDayOfYear);
        System.out.println("Period Format= "+period);
        System.out.println("Months remaining in the year= "+period.getMonths());
    }
}
```

```
Year 2020 is Leap Year  true
Today is before 01/01/2015  false
Current Time=2020-03-20T15:07:05.814507
10 days after today will be 2020-03-30
3 weeks after today will be 2020-04-10
20 months after today will be 2021-11-20
10 days before today will be 2020-03-10
3 weeks before today will be 2020-02-28
20 months before today will be 2018-07-20
First date of this month= 2020-03-01
Last date of this year= 2020-12-31
Period Format= P9M11D
Months remaining in the year= 9
```

