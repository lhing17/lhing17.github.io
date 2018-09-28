layout: post
title:  "就从Java8开始吧（五）新日期和时间API"
categories: Java
tags:  Optional Java8
author: G. Seinfeld

* content
{:toc}

### 闲话
> “明天我们一起去吃饭好不好？”“好啊，那我们就1533810697一起去吧！”

这种对话显然不会在生活中发生，首先这不是人类描述时间的方式，其次你脑子也不可能这么快算出来现在的时间戳。当然最关键的，如果你这么说话，会被人锤死。

然而，虽然正常人不会这么说话，java却会。至少在java8出现之前，java就是这么说话的。准确点说，java8之前的日期和时间API就是这么设计的。

java8之前，java经历了两个版本的日期和时间API，其中第一版是java1.0时代出现的以java.util.Date类为核心的API，第二版是java1.1时代的java.util.Calendar类。第一代API由于太难用，只存在了一个版本就废弃了大部分方法。第二代虽然用了很多版本，但是它本身一些设计缺陷一直饱受诟病，因此在java8的新日期API出现之前，出现了一些设计优秀的日期API，作为JDK中API的替代品，比如Joda-time。

由于日期和时间操作是一门编程语言最常规的操作之一，在java8设计时，设计师们决定对日期API进行改良，于是有了今天的java.time包。

不同于过去版本的日期API，新版本API有以下几方面的优势：
- 不可变性，以及随之而来的线程安全
- 人类阅读和机器处理的日期和时间分离
- 优秀的时区处理
- 更清晰的编码风格

### API干货
让我们具体来看一下java8的新日期和时间API是怎么组织的，以下API按照使用频率进行排序：
#### 1. Instant
Instant代表的是时间戳，是一个准确的时间，它内部维护了一个long类型的秒数（自格林威治时间1970年1月1日零时起的秒数）和一个int类型的纳秒数，实际的时间应该是两者拼接起来的结果。Instant没有公有化的构造器，只能通过工厂方法进行构造。常用的工厂方法有：
```java
// 返回一个当前时间的时间戳，注意时间戳是和时区无关的
public static Instant now();
// 利用（自格林威治时间1970年1月1日零时起的）秒数进行构造
public static Instant ofEpochSecond(long epochSecond);
// 与上面方法相似，多了一个纳秒的校正
public static Instant ofEpochSecond(long epochSecond, long nanoAdjustment);
// 利用（自格林威治时间1970年1月1日零时起的）毫秒数进行构造
public static Instant ofEpochMilli(long epochMilli);
```

构造出时间戳后，可以对其进行偏移操作，可以计算两个时间戳之间的间隔，也可以比较两个时间戳的先后顺序。

```java
// 在原时间戳上增加一个时间偏移量
public Instant plus(TemporalAmount amountToAdd);
// 在原时间戳上减少一个时间偏移量
public Instant minus(TemporalAmount amountToSubtract);
// 判断一个时间戳是否在给定时间戳之后
public boolean isAfter(Instant otherInstant);
// 判断一个时间戳是否在给定时间戳之前
public boolean isBefore(Instant otherInstant);
```

计算时间戳间隔的方式以及时间偏移量的构造都依赖于下一小节介绍的Duration，我们一会再看。通过上面的API，我们可以直观清晰地看到，加减的方法进行了分离，不用再像Calendar时代的日期操作一样加一个负数来实现时间的减少了，这样可读性和便于理解方面都得到了较大的提升。

#### 2. Duration

Duration和Instant非常像，也是在内部维护了一个long类型的秒数（自格林威治时间1970年1月1日零时起的秒数）和一个int类型的纳秒数，区别在于它表征的是两个时间戳的时间间隔。可以手动构造一个时间间隔，用于对时间戳进行偏移，也可以计算两个时间戳之前的时间差，用于反应某种执行时间。

```java
/* 构造Duration的工厂方法 */
// 以天为单位构造时间间隔
public static Duration ofDays(long days);
// 以小时为单位构造时间间隔
public static Duration ofHours(long hours);
// 以分为单位构造时间间隔
public static Duration ofMinutes(long minutes);
// 以秒为单位构造时间间隔
public static Duration ofSeconds(long seconds);
// 以秒为单位构造时间间隔，增加纳秒校正
public static Duration ofSeconds(long seconds, long nanoAdjustment);
// 以毫秒为单位构造时间间隔
public static Duration ofMillis(long millis);
// 以纳秒为单位构造时间间隔
public static Duration ofNanos(long nanos);
// 以任意时间单位构造时间间隔
public static Duration of(long amount, TemporalUnit unit);
// 通过时间量为单位构造时间间隔
public static Duration from(TemporalAmount amount);
// 从字符串中解析出时间间隔
public static Duration parse(CharSequence text);
// 计算两个时间点之间的时间间隔
public static Duration between(Temporal startInclusive, Temporal endExclusive);
```

 构造之后，可以对时间间隔做各种加减乘除操作，由于API数量太多，这里仅举几个例子：

```java
// 增加若干秒
public Duration plusSeconds(long secondsToAdd);
// 减少若干分钟
public Duration minusMinutes(long minutesToSubtract);
// 做乘法
public Duration multipliedBy(long multiplicand);
// 做除法
public Duration dividedBy(long divisor);
// 取绝对值
public Duration abs();
```

注意时间间隔是一个有方向的量，即可以是正数也可以是负数。因此，如果想获得正数的时间间隔，可以对结果取绝对值。

以下程序可以计算程序运行的时间：

```java
Instant start = Instant.now();
// 一些要执行的程序
...
Instant end = Instant.now();
// 开始时间在前，结束时间在后
Duration duration = Duration.between(start, end);
// 打印程序运行的毫秒数
System.out.println(duration.toMillis());
```

#### 3. LocalDate、LocalTime、LocalDateTime三兄弟

了解了时间戳和时间间隔，我们需要一些真正来描述时间的类。作为一个现实生活中的人，我们能够真切感受到和使用到的时间就是我们钟表上显示的时间，也就是本地时间。LocalDate、LocalTime以及LocalDateTime这三兄弟就是为了描述这种时间而设计的。它们不考虑时区的因素，也不太用于精细时间的计算，只是为了表征一个人类能够轻松理解的时间。它们同样通过工厂方法来构造，这里举一些例子：

```java
/* 构造LocalDate */
// 根据年月日来构造日期，注意这里的月份终于是从1开始的了（1-12分别代表1-12月）
public static LocalDate of(int year, int month, int dayOfMonth);
// 根据年份和儒略日来构造日期
public static LocalDate ofYearDay(int year, int dayOfYear);

/* 构造LocalTime */
// 当前时间
public static LocalTime now();
// 使用时分秒构造时间
public static LocalTime of(int hour, int minute, int second);
// 使用一天中的秒数来构造时间
public static LocalTime ofSecondOfDay(long secondOfDay);

/* 构造LocalDateTime */
// 使用年月日时分秒来构造日期和时间
public static LocalDateTime of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond);

/* 三兄弟间的转换 */
// LocalDate类中，将LocalDate和LocalTime拼接成LocalDateTime
public LocalDateTime atTime(LocalTime time);

/* 时间的偏移 */
// LocalDate
public LocalDate plusYears(long yearsToAdd);
public LocalDate minusMonths(long monthsToSubtract);

// LocalTime
public LocalTime plusMinutes(long minutesToAdd);
public LocalTime minusHours(long hoursToSubtract);

// LocalDateTime
public LocalDateTime plus(TemporalAmount amountToAdd);
public LocalDateTime minusNanos(long nanos);
```

#### 4. 日期格式化

Java 8 以前我们用的日期格式化的类是`java.text.SimpleDateFormat`，Java 8 提供的日期格式化类是`java.time.format.DateTimeFormatter`，DateTimeFormatter可以通过以下的方法构造：

```java
// 利用格式化字符串构造格式化对象
public static DateTimeFormatter ofPattern(String pattern);
```

时间转字符串：

```java
LocalDate date = LocalDate.now();
String text = date.format(formatter);
```

字符串转时间：

```
String text = "2018-09-27";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
LocalDate parsedDate = LocalDate.parse(text, formatter);
```

新时间API还提供了一些与时区相关的类，大家可以自行翻阅相关API。

### 设计思路

1. 不可变类的构造

- 类设计为final、所有成员变量私有，并且加上final修饰
- 构造器私有化，使用静态工厂方法调用私有构造器，用于构造对象
- 在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝

2. 新日期API中，所有的类都是不可变且线程安全的。
3. 新日期API设计中，特别强调了代码的可读性和执行效率。在当前条件下，人的时间远贵于机器的时间。

### 最佳实践

1. 在新的API设计时，优先使用新版日期API。
2. 历史遗留代码的API，如果不好转成新版日期类，不必强行转换，否则可能反而降低效率。

### 练习与思考

1. 如何使用新日期API计算一段代码的执行时间？
2. 计算2010年1月10日与2018年12月21日的天数差。
3. 计算00:30:28与19:38:52秒之间的秒数差。
4. 利用新日期API实现一个定时器的功能。
5. 设计一个不可变类。