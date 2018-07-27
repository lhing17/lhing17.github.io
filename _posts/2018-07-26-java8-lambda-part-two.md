---
layout: post
title:  "就从Java8开始吧（二）lambda表达式和方法引用"
categories: Java
tags:  lambda Java8
author: G. Seinfeld
---

* content
{:toc}
书接上文，在[第一篇](https://www.jianshu.com/p/3ce65e13d967)中，我们介绍了lambda表达式的来历和一些简单的用法。这一篇我们作一点补充，以便加深大家对lambda表达式的理解。
首先回顾上一篇的重点（敲黑板）：

>**lambda表达式是Java8对于函数式编程思想的一种实现方式，它的本质是函数式接口的实现类；函数式接口是指只有一个需要实现的抽象方法的接口，函数式接口存在的意义是包装要作为参数或返回值的方法。**

这一篇我们说一说最常用的函数式接口，以及对应的lambda表达式的写法。先补充一个小概念，方法体只有一行代码（没有{}包围）的lambda表达式称为表达式型lambda表达式（expression lambda），方法体有多行代码的称为语句型lambda表达式（statement lambda）
最常用的函数式接口有四个：
消费型接口、供给型接口、断言型接口以及功能型接口，对应的Java接口名分别为Consumer、Supplier、Predicate以及Function，这四个接口都提供泛型的支持，下面分别来说一说。

**一、消费型接口Consumer**

消费型接口包装了一个accept方法，它接收一个泛型参数并进行处理，没有任何返回值。通俗点说，消费型接口就是“只吃不吐”的类型。Consumer接口是四个接口中唯一一个明确接受副作用的接口，也就是说，使用Consumer接口时，输入参数的状态可能发生改变。Consumer接口通常用于动态处理一些数据或业务。比如Java8中集合遍历的forEach方法：
```Java
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```
利用Consumer接口，我们可以很方便的对集合中每一个元素进行操作。这里使用到了default方法，它是另一项1.8特性，我们在后续篇幅中会有介绍。Consumer接口对应的lambda表达式长这样：
```Java
t -> doSomething(t)
```
这里注意尽管把方法体进行封装，以便保持lambda表达式的简洁性和可读性。

**二、供给型接口Supplier**

供给型接口包装了一个get方法，它不接收参数，返回一个泛型对象。供给型接口是一群只懂付出、不图回报的奉献者，一般用来生成结果。Supplier接口对应的lambda表达式看起来是这样：
```Java
() -> generate()
```
注意上面的generate()应该有一个返回值，但是在lambda表达式的方法体中，如果只有一行代码的话，不需要有return关键字以及分号，写的话会产生编译错误。

**三、断言型接口Predicate**

断言型接口包装了一个test方法，它接收一个泛型参数，返回一个boolean类型的参数。这类接口一般用于判断对象是否满足某种约束条件。Predicate接口对应的lambda表达式应该类似于：
```Java
t -> check(t)
```
其中check是一个返回boolean类型的方法，用于对t值进行校验。

**四、功能型接口**

功能型接口包装了一个apply方法，它接收一个泛型参数，返回另一个泛型值。这类接口更为通用，可以用于映射对象，如Optional的map方法（注：Optional也是Java8的一项特性，后面会讲到）。
Function接口对应的lambda表达式为：
```Java
t -> apply(t)
```
其中apply是一个返回另一个泛型U类型的方法。

说完常见的函数式接口和对应的lambda表达式，我们有请今天的另一位主角：方法引用（method reference）。方法引用是一种语法糖，它是对lambda表达式的进一步简化，当然这种简化是有条件的，并不是所有lambda表达式都能简化成方法引用的形式。那么方法引用究竟长什么样呢，让我们一起来揭开它神秘的面纱。

在使用lambda表达式时，我们发现，有些lambda表达式只有一行代码，并且仅仅是调用了一个已存在的方法。这种情况下，我们可以使用方法引用来进一步简化。举一个字符串比较的例子，在字符串数组排序进程中，我们通常调用Arrays.sort方法进行排序，Arrays.sort方法有一个重载方法，传入Comparator参数以便于自定义排序的规则：
```Java
Arrays.sort(stringsArray,(s1,s2)->s1.compareToIgnoreCase(s2));
```
上面的lambda表达式传入了一个无视大小写的字符串比较策略，它仅仅是对现有的compareToIngoreCase方法进行了调用。那么它可以使用方法引用简化如下：
```Java
Arrays.sort(stringArray, String::compareToIgnoreCase);
```

方法引用引用了::符号进行表示，具体来说，方法引用有四种形式：


| 类型 | 示例 |
| :---- | :---- |
| 引用静态方法 | ContainingClass::staticMethodName |
| 引用某个对象的实例方法 | containingObject::instanceMethodName |
| 引用某个类型任意对象的实例方法 | ContainingType::methodName |
| 引用构造器 | ClassName::new |

注意上面的大小写，大写代表的是类名，小写代表的是对象名。还要注意几种情况后面都不带括号。

引用静态方法比较好理解，比如
```java
(o1, o2) -> Math.min(o1, o2)
```
可以简化为
```java
Math::min
```
引用某个对象的实例方法一般指的是lambda表达式外的参数，如声明了
```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
```
那么lambda表达式
```java
s -> sdf.parse(s)
```
就可以简写成
```java
sdf::parse
```
引用某个类型任意对象的实例方法一般指的是lambda表达式内的参数，如上面的例子
```java
Arrays.sort(stringsArray,(s1,s2)->s1.compareToIgnoreCase(s2));
```
简化为
```java
Arrays.sort(stringArray, String::compareToIgnoreCase);
```
引用构造器就更简单了，比如有个Person类，有个构造器
```java
Person(String name){...}
```
此时lambda表达式
```java
name -> new Person(name)
```
可以简化为
```java
Person::new
```

晕了吧，可以从头再看一遍嘛。童鞋们，下课。