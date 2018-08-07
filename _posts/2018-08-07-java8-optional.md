---
layout: post
title:  "就从Java8开始吧（四）唠一唠Optional"
categories: Java
tags:  Optional Java8
author: G. Seinfeld
---

* content
{:toc}

### 闲话

这次咱们来唠一唠Optional。说Optional之前，要先提一个Java程序员几乎每天都会被它折磨的东西：NPE。NPE——NullpointerException——空指针异常，毫无疑问是Java程序员每天见到最多的异常，没有之一。俗语有云：“一日三空”，今天你空指针了吗？

避免空指针异常也成为了Java程序员的一项最基本的素质。我们知道，只需要一个简单的if not null的判断，就可以完全规避掉空指针的风险。但是空指针又无孔不入，有时是我们疏忽忘记加判断，有时是涉及的变量和调用步骤太多，层层嵌套判断过于繁琐，心想索性还不如直接不判断。于是在你大意的时候，空指针又来了。

为了解决头疼的空指针问题，Java8派出了救星，也就是我们今天的主角：Optional。Optional是专门为了避免空指针而设计的，但它并不是Java8的新鲜事物——谷歌的guava库很早（10.0版本）就引入了Optional类。Java8的设计者一看这玩意好用，于是就拿来主义了。然而正统还是正统，即使是抄的它也是正统，在当今的IDE里，如果你还在坚持使用guava库的Optional类，编译器会毫不留情地给你一条编译警告，告诉你应该用JDK里的Optional类。

### API干货
闲话少叙，让我们一起来看一下Optional是个什么东西。Optional是java.util包中的一个类，它的本质是一个一元容器，可以包含一个非空值或一个null值，它设计了一系列API来帮助程序员规避空指针风险。

Optional类有一个泛型参数，表明它是一个可以容纳任意类型参数的容器。它没有公有的构造器，只能通过静态的工厂方法进行创建：
```java
// 只能传入非空值的静态方法
public static <T> Optional<T> of(T value);
// 可以传入空值的静态方法
public static <T> Optional<T> ofNullable(T value)
```
其中of方法只可以传入非空值，ofNullable方法则可以传入空值，创建以后返回一个包含传入值的Optional对象，之后就可以利用API对该对象进行操作。

Optional类有一个get方法，用于获取容器中的非空值，如果Optional容器中装了一个null值，get方法将抛出NoSuchElementException异常。因此get方法需要配合isPresent方法使用，后者返回一个boolean值，用于判断容器中是否包含有非空值。介绍完这几个方法以后，我们就可以对Optional对象进行一些常规操作了，例如想打印某个字符串的子字符串：
```java
String s = null;
// 构造Optional容器
Optional<String> o = Optional.ofNullable(s);
if (o.isPresent()){
    // 从容器中取出对象
    String ss = o.get();
    // 打印子字符串
    System.out.println(ss.subString(0));
}
```

看到这里，有的同学会问了，这和原来的判空有什么区别？
```java
String s = null;
if (s != null ) {
    System.out.println(s.subString(0));
}
```

答案是没有区别。如果Optional只有以上功能的话，它只是普通的把戏，称不上魔术。真正让它化腐朽为神奇的是下面提到的API。
```java
// 将一种Optional对象映射为另一种Optional对象
public<U> Optional<U> map(Function<? super T, ? extends U> mapper);
// 如果容器中包含的是一个空值，那么返回传入的值
public T orElse(T other);
public void ifPresent(Consumer<? super T> consumer);
```
map是将Optional对象转换为另一种Optional对象，也就是相当于在Optional的包裹下进行了方法的调用，利用map可以规避链式调用中的空指针问题。orElse方法则是对map方法的一个补充，在map链式调用的任何一个环节中返回了空值，都会走到orElse方法，返回传入的值。举个例子，作为Java程序员，大家一定都写过形如
```java
a.getB().getC().getD()
```
的链式调用，当然，老大哥们会告诉你这种写法违背迪米特法则，这是后话了。先不考虑迪米特法则，我们发现在这种链式调用中，只要有一个环节返回null，就会产生空指针异常。层层嵌套的if判断又过于繁琐，这时就应该考虑使用Optional类来处理这种情况了：
```java
Optional.ofNullable(a)
        .map(A::getB)
        .map(B::getC)
        .map(C::getD)
        .orElse(E);
```
这样在整个调用中出现null，整个表达式的值就为E，否则表达式的值就是c.getD()。
ifPresent是另一个实用的方法，它传入一个消费者对象，如果Optional容器中值非空，执行消费者对象的方法，否则不做任何事情，这样就规避了空指针的风险。

### 设计思路
说到这里，可能大家还是没有充分认识到Optional的好处，总觉得用if判断也可以达到相同的效果。如果语法的简洁不足以吸引你去使用Optional，那么我告诉大家，Optional最大的用处在于强迫API的调用者去考虑空指针的风险。不推荐将Optional类型的对象作为类的属性或者方法的参数，甚至多数情况下也不推荐自己构造Optional。最合理的用法应该是作为API方法的返回值，以便强制调用API的人去考虑API中可能存在的空指针风险。例如Stream的reduce方法：
```java
T reduce(T identity, BinaryOperator<T> accumulator);
Optional<T> reduce(BinaryOperator<T> accumulator);
```
reduce方法有两个重载方法，一个是手动传入种子，一个是使用Stream中第一个元素作为种子。我们可以看到，这两个重载方法一个返回泛型对象，一个返回封装泛型对象的Optional容器。这是因为手动传入种子的情况不存在空指针的风险，而使用Stream中第一个元素作为种子则需要考虑可能的空指针风险。大家可以学习一下这个思路，在以后的API设计中不给调用者留坑。顺便说一下，Java规范中明确指出规避空指针风险是调用者的责任，因此在调用API的时候，要充分考虑API的空指针风险，不要过度依赖API本身的设计，即所谓的防守式编程。

### 总结一下
>- Optional是设计用于规避空指针风险的一元容器类
>- Optional容器中可以存入一个非空值或一个null值
>- Optional类的get方法需要结合isPresent方法使用
>- Optional类的map方法和orElse方法结合可以优雅地实现链式调用。
>- 不推荐将Optional作为类的属性或方法的参数，最好是作为返回值以强制调用者考虑方法的空指针风险

以上，祝大家早日摆脱空指针的苦海~