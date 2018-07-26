---
layout: post
title:  "就从Java8开始吧（一）lambda表达式详解"
categories: Java
tags:  lambda Java8
author: G. Seinfeld
---

* content
{:toc}
一直想写一套技术的文章，苦于不知从何落笔。今天得空从忙碌的敲代码中抽身出来，就从Java8开始说起吧。

在我看来，Java至今有两个翻天覆地的版本，一个是5，另一个就是8。Java8发布于2014年，至今已经四年多了，伴随着9和10的发布，8也不能算是多新的版本了。那么Java8的十大“新”特性如今也应该是各个程序员基本技能的一部分了。时至今日，很多软件也纷纷把jdk的最低要求设置到了8，程序员们也没有任何理由使用低于jdk8的java进行开发了。

那么，Java8提供了哪些特性，可以称得上“翻天覆地”呢？让我们一起来看一下吧。

首先，最最主要的，就是从语言层面引入了函数式编程的思想。很多脚本语言对函数式编程有着很好的支持，比如javascript和python，函数（Java中叫方法，以下可以将函数和方法作为同义词）可以像其他类型的数据一样作为函数的参数或者返回值，这样我们可以轻松地实现策略模式。比如在自定义排序函数中，我们可以将排序的策略作为一个参数传递给负责排序的函数。在Java8之前，Java对函数式编程的支持就显得十分笨重了。由于Java的方法都依赖于类，因此想要将方法作为参数或者返回值，必须用类将方法进行包装（wrap），然后将类的对象作为参数或者返回值进行传递。这样的实现无论从语法书写上，还是从阅读上，都显得不那么优雅。因此，Java的语言设计师们决定引入对函数式编程更优雅的支持。那么，过程中，设计师们做了很多考量，比如像其他语言一样，引入函数这样的数据类型。但是，受到兼容历史版本等限制的影响，最终设计师们采用了一种比较折衷的方法，这就是我们所说的lambda表达式。

下面隆重有请我们今天的主角——lambda表达式登场。lambda表达式是Java8引入的一项新语法，用->符号实现对函数式编程的支持。第一眼看到lambda表达式时，我觉得Java不那么像Java了，但是经过一段时间的使用体会，我感受到了这一项新语法引入的强大之处，也逐渐接受了Java大军中的这样一个新伙伴。为什么叫lambda表达式呢？lambda是希腊字母λ的英文拼写，在各个语言中，均有代表匿名函数之意。Java中也沿用了这一定义方式，将新的语法称为lambda表达式。lambda表达式的书写很简单，如：
```
x, y -> x + y
() -> doSomeThing()
(int x, String s) -> x < s.length()
```
上述表达式表达的含义可就没那么简单了。它其实是一种语法糖，代表一个被接口包装的方法。如上面的第一行代码，表达的是一个二元求和的方法。它大致相当于：
```
interface Calc {
    int sum(int x, int y)
}
class CalcImpl implements Calc{
    int sum(int x, int y){
        return x + y;
    }
}
```
但是在lambda表达式中，x和y的类型是根据实际情况进行推断的。当然也可以显式的指定x和y的类型。
看到这里各位可能有点蒙。没关系，上面只是为了让大家看到lambda表达式语法上的简洁性。下面我们具体来说一说lambda表达式是怎么一回事。我们还是从上面排序的例子说起。比如现在有一个字符串数组:
```
{"abc", "abcde", "abbb", ....}
```
我们要对数组进行自定义排序，比如按照字符串长度进行排序。我们知道Java提供了Arrays.sort方法进行数组排序，重载方法中有一个方法传递了一个Comparator接口的对象用于实现自定义排序，在Java8以前，我们会这么写：
```
String[] array = {"abc", "abcde", "abbb", "bbccd"};
Arrays.sort(array, new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
        return o1.length() - o2.length();
    }
});
```
即使用一个匿名内部类来实现Comparator接口。（不熟悉这一段代码的童鞋可以回去翻一翻Java基础哦）。引入lambda表达式以后，我们可以这么写：
```
String[] array = {"abc", "abcde", "abbb", "bbccd"};
Arrays.sort(array, (o1, o2) -> o1.length() - o2.length());
```
可以看到，lambda表达式用一行代码代替了上面六行代码。lambda表达式由参数列表，箭头符号（->）和方法体三部分组成。参数列表与接口的compare方法参数列表相同，类型可以自动推断，方法体代表对这个方法的实现。方法体使用{}包围，如果只有一行代码，{}可以省略。
为了进一步理解lambda表达式的原理，这里要先引用函数式接口的概念。我们上面提到了，Java的方法依赖于类，这一点即使在Java8中也没有改变。因此想要传递一个方法作为参数或者返回值，我们还是要传递一个类（或者接口）的对象。函数式接口就是这样的接口，它的内部只有一个未实现的抽象方法，也就是说我们使用这个接口其实就是为了使用这个未实现的方法，然后对它进行实现。例如：
```
@FunctionalInterface
interface Wrapper{
    void method(int param);
}
```
Wrapper接口的唯一作用就是包装method方法，将其作为使用方法的参数或者返回值进行传递。上面的@FunctionalInterface注解是可选的，它可以在编译阶段检查接口是否符合函数式接口的要求，如果不符合，则会编译失败。
现在我们可以清晰地看到，从本质上讲，lambda表达式就是函数式接口的匿名实现类。凡是参数中传递函数式接口对象的地方，我们都可以使用lambda表达式进行替换。例如线程的创建：
```
// 使用匿名内部类创建线程
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        doSomeThing();
    }
};
new Thread(runnable).start();

// 使用lambda表达式创建线程
Runnable runnable = () -> doSomeThing();
new Thread(runnable).start();
```
再如，集合的遍历：
```
/*
 * Collection forEach循环
 */
List<Integer> list = Lists.newArrayList(1, 2, 3, 4, 8, 7, 6, 5);
// fori循环
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}

// foreach循环
for (int ele : list) {
    System.out.println(ele);
}

// lambda表达式
list.forEach(ele -> System.out.println(ele));
```
好了，先到这里吧。下一篇我们将分析lambda表达式的更多使用场景，以及如何使用方法引用进一步简化lambda表达式。