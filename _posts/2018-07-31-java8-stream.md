---
layout: post
title:  "就从Java8开始吧（三）说一说Stream"
categories: Java
tags:  Stream Java8
author: G. Seinfeld
---

* content
{:toc}

前两讲我们聊了一聊Java8的lambda表达式，有的同学一定会问，lambda表达式仅仅是一种语法糖，仅仅起到了美化代码的作用么？

答案是也不是。说是是因为它的的确确只是一种语法糖，换句话说，Java8中使用lambda表达式能实现的东西，在Java7及之前的版本中几乎一定可以实现。说不是是因为有了lambda表达式，API的设计得到了更充分的发挥空间，极大地提升了编程的效率和可读性，这样各种逆天的API才能如雨后春笋般出现。

说起逆天的API，就不得不提起jdk8中新加入的Stream机制，它可以说是Java API中的魔术师，将lambda表达式的价值发挥得淋漓尽致。Stream称为流式计算，它不同于文件读写流（InputStream、OutputStream）或是xml解析流，它是对集合（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作（aggregate operation），或者大批量数据操作 (bulk data operation)。

在进一步了解Stream之前，我们首先需要了解一下什么是聚合操作。简单点说，聚合操作就是把一堆数通过某种统计计算变成一个数。比如求一组数据的个数、平均值、最小值、最大值、和、标准差之类的操作。放到具体业务中，可能是计算某个班级的平均分、某个商场中最便宜的商品、某个公司最年长的员工等等。在Stream出现之前，Java对于数据聚合操作的能力是很弱的，要么要求程序员自己编写大量的代码进行数据处理，要么依赖于关系型数据库的聚合函数进行操作。Stream出现之后，Java自带的API终于也能高效处理数据了。

那么究竟什么是流呢？首先我们看一下流的概念。流不是数据结构，不保存数据，它是一种特殊的迭代工具，它是关于算法和计算的，它可以对数据进行转换、映射、过滤和筛选，并通过一系列操作获取到想要的结果。流是怎么高效处理数据的呢？据说一个成功的魔术有三个步骤：以虚代实、偷天换日、化腐朽为神奇（引自《致命魔术》）。Stream也是通过三个类似的步骤完成了对于数据的高效处理。

一、以虚代实————数据源转换成流
这一步是关于流的创建的。我们拿到的数据源通常是一个可迭代的数据结构，如集合或数组。如果数据源是集合，我们可以利用集合的stream()方法进行流的创建，如：
```java
List<Integer> list = Lists.newArrayList(1, 3, 5, 7, 9);
Stream<Integer> stream = list.stream();
```
如果数据源是数组，我们可以利用Stream类提供的工具方法进行流的创建，如：
```java
String[] array = {"1", "3", "5", "7", "9"};
Stream<String> stream = Stream.of(array);
```
也可以使用Arrays工具类中的stream方法进行创建，如：
```java
String[] array = {"1", "3", "5", "7", "9"};
Stream<String> stream = Arrays.stream(array);
```
注意，Stream包含一个泛型参数，表明流处理的是哪种类型的数据，Stream的泛型参数应该与数据源的类型一致。

二、偷天换日————数据的转换
这一步是从一个流转换成另一个流的过程，由于转换结果还是流，因此这一步可以多次进行，这也是流式计算理念的核心。
流的数据转换操作（也叫中间操作，intermediate）主要包括map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered等。
这里介绍几种常见的操作，其他操作各位读者可以自行翻阅API文档。

- map
这里的map不是地图，有点类似于集合框架里的map，是映射的意思，可以将流中的元素按照某种规则统一映射成另一种元素，例如将字符串列表中所有字符串变为大写：
```java
List<String> list = Lists.newArrayList("abc", "def", "ghi");
// 初始生成的流
Stream<String> initial = list.stream();
// 转化为所有元素均为大写字母的流
Stream<String> mapped = initial.map(s -> s.toUpperCase());
// 使用方法引用来表达
// Stream<String> mapped = initial.map(String::toUpperCase);
```
这里映射的方案是通过功能型接口（Function，见上一篇）来表达的，可以使用lambda表达式或者方法引用来实现功能型接口。通过这个例子我们也可以看出lambda表达式（以及方法引用）对于代码简洁度和可读性的重要提升。如果不使用lambda表达式，我们就要通过匿名内部类这种繁琐的代码来实现功能性接口了。

- filter
filter顾名思义，就是对流里的元素进行筛选。例如筛选所有大于10的数字：
```java
List<Integer> list = Lists.newArrayList(1, 100, 30, 5, 18, 9, 6);
// 初始生成的流
Stream<Integer> initial = list.stream();
// 只保留大于10的元素，这时流里的元素包括100, 30, 18
Stream<Integer> filtered = initial.filter(i -> i > 10); 
```
filter方法中过滤的方案是通过断言型接口（Predicate）来表达的，它接收一个参数并返回一个boolean值。如果返回结果为true，表明元素符合要求，应该保留；反之元素被过滤掉。

- sorted
将流中的元素进行排序，有一个无参方法和一个接受Comparator类型参数的重载方法。无参方法按照自然排序（即实现Comparable接口的类的默认排序）进行排序，如果元素所属的类型没有实现Comparable接口，则会抛出ClassCastException（类型转换异常）。有参数的重载方法则可以按照Comparator提供的策略进行排序，Comparator类型同样可以使用lambda表达式进行实现。例如：
```java
List<Integer> list = Lists.newArrayList(1, 100, 30, 5, 18, 9, 6);
// 初始生成的流
Stream<Integer> initial = list.stream();
// 无参方法实现自然排序
// Stream<Integer> sorted = initial.sorted();
// 有参方法实现自定义排序
Stream<Integer> sorted = initial.sorted((o1, o2) -> o2 - o1);
```

- 并行操作 parallel、 sequential、 unordered
Stream的一个强大之处在于支持并行操作，Stream通过并行流支持并行操作，并行流能够借助多核处理器并行执行代码，这样可以显著提高性能。并行流的API简单可靠，在一定程序上规避了多线程并发编程的复杂性。
```java
List<Integer> list = Lists.newArrayList(1, 100, 30, 5, 18, 9, 6);
// 初始生成的流
Stream<Integer> initial = list.stream();
// 将流转换为并行流
Stream<Integer> paralleled = inital.parallel();
// 判断流是否为并行流
boolean b = paralleled.isParallel();
System.out.println(b);
```
关于怎么使用并行流进行编程，限于篇幅这里就先不展开了，回头有机会我们单独开辟一个章节来介绍。

三、化腐朽为神奇————数据的聚合

作为一个成功的魔术师，光有前两个步骤是不够的，最重要的一个步骤就是“化腐朽为神奇”——把流转换成我们想要的结果。在流的操作中，这一个步骤被称为数据的聚合，也叫流的中止操作（termination）。常见的流中止操作包括forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator。注意，流做了中止操作后，就不能再进行其他操作了，否则会报IllegalStateException异常（java.lang.IllegalStateException: stream has already been operated upon or closed）。

- reduce
reduce是最著名的流中止操作，它和map并称和map-reduce（因作为谷歌搜索引擎的核心算法而出名）。它的主要作用是对Stream元素进行依次聚合。它提供一个种子（或者把第一个元素作为种子），然后将种子和第一个元素按某种规则（比如加减乘除）进行聚合，得到的结果再与第二个元素按照相同规则进行聚合。从某种意义上讲，上面提到的mix、max、sum、average等都是特殊的reduce。例如：
```java
List<Integer> list = Lists.newArrayList(1, 100, 30, 5, 18, 9, 6);
// 初始生成的流
Stream<Integer> initial = list.stream();
// 利用reduce进行求和操作
Integer sum = initial.reduce(0, (a, b)->(a + b));
```

- forEach
forEach是java8提供的新版本遍历，与Collection类中的forEach用法相同，即对流中的每一个元素进行某种操作。注意在并行流中，遍历操作不能够保证执行的顺序。
```java
List<Integer> list = Lists.newArrayList(1, 100, 30, 5, 18, 9, 6);
// 自然排序后取前5个元素
Stream<Integer> stream = list.stream().sorted().limit(5);
// 打印流中某个元素
stream.forEach(System.out::println);
```

- collect
collect意为收集操作，它和其他聚合操作略有不同，它不是将流中所有元素聚合成一个值，而是收集为可以查看的结果。例如将流重新转换为List。collect有两个常用的重载方法，一个是基础方法：
```java
<R> R collect(Supplier<R> supplier,
            BiConsumer<R, ? super T> accumulator,
            BiConsumer<R, R> combiner);
```
它需要传递三个参数：一个是供给者，用于产生最后结果的容器，如生成一个新的ArrayList；一个是累加器，用于将Stream中的元素累加到一起，一个是组合器，用于将累加后的结果进行组合添加到容器中。例如：
```java
    // 取大于5的元素
    Stream stream = Stream.of(1, 100, 30, 5, 18, 9, 6).filter(p -> p > 5);
    // 供给者用于提供ArrayList，累加器用于将Stream中的item累加到list中，组合器用于将累加的结果组合到一起
    List result = stream.collect(() -> new ArrayList<>(), (list, item) -> list.add(item), (one, two) -> one.addAll(two));
```
基础方法用起来比较麻烦，如果只是想将Stream转换为List的话，可以使用collect的重载方法
```java
<R, A> R collect(Collector<? super T, A, R> collector);
```
可以使用Collectors工具类中的静态方法去构造collector:
```java
     // 取大于5的元素
    Stream stream = Stream.of(1, 100, 30, 5, 18, 9, 6).filter(p -> p > 5);
    // 使用Collectors的静态方法构造collector
    List result = stream.collect(Collectors.toList());
```
其他像min、max、count等方法的使用较为简单，这里不再详细展开了。
总结一下，Stream具有以下特性（敲黑板）：
>- Stream不是数据结构，也不会修改源数据结构中的数据
>- Stream操作的参数均为函数式接口（无参数的除外）
>- Stream的三个步骤：生成流、转换流、聚合
>- Stream具有并行的能力，可以取代多线程数据处理
>- Stream可以是无限的
