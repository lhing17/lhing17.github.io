---
layout: post
title:  "就从Java8开始吧（五）新日期和时间API"
categories: Java
tags:  Optional Java8
author: G. Seinfeld
---

* content
{:toc}

### 闲话
> “明天我们一起去吃饭好不好？”“好啊，那我们就1533810697一起去吧！”

这种对话显然不会在生活中发生，首先这不是人类描述时间的方式，其次你脑子也不可能这么快算出来现在的时间戳。当然最关键的，如果你这么说话，会被人锤死。

然而，虽然正常人不会这么说话，java却会。至少在java8出现之前，java就是这么说话的。准确点说，java的日期和时间API就是这么设计的。

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
Instant代表的是时间戳，是一个准确的时间，它内部维护了一个秒数（自格林威治时间1970年1月1日零时起的秒数）和一个纳秒数，实际的时间应该是两者拼接起来的结果。Intant没有公有化的构造器，只能通过工厂方法进行构造。常用的工厂方法有：
```java
// 返回一个当前时间的时间戳，注意时间戳是和时区无关的
public static Instant now();
// 利用（自格林威治时间1970年1月1日零时起的）秒数进行构造
public static Instant ofEpochSecond(long epochSecond);
// 与上面方法相似，多了一个纳秒的校正
public static Instant ofEpochSecond(long epochSecond, long nanoAdjustment);
// 利用（自格林威治时间1970年1月1日零时起的）毫秒数进行构造
public static Instant ofEpochMilli(long epochMilli);

Instant instant = Instant.now();
```

#### 2. Duration

#### 3. LocalDate、LocalTime、LocalDateTime三兄弟


### 设计思路
- 类设计为final、所有成员变量私有，并且加上final修饰
- 构造器私有化
- 在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝


### 最佳实践

### 练习与思考
