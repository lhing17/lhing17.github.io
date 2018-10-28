---
layout: post
title:  就从Java8开始吧（六）接口默认方法和静态方法
categories: Java
tags:  Java8
author: G. Seinfeld
---

* content
{:toc}

### 闲话
这一节我们聊一聊接口的默认方法和静态方法。一切要从接口开始说起。接口是泛指一种实体把自己提供给外界的抽象化物，它最明显的特征是把内部操作和实现方式隐藏，只保留与外部最直接最简单的沟通。举个例子，我们每天要吃饭，通过食物来摄取营养，这时我们和食物之间的接口就是我们的嘴巴，我们只需要告知外界我们通过嘴巴吃食物，至于在胃里怎么消化，在肠里怎么吸收，都属于身体的实现细节，没有必要暴露给外界。

在Java语言中，接口是一系列抽象方法的集合，通过inferface关键字来声明。类通过实现（implements）接口的方式，来实现接口中的抽象方法。在Java8之前，接口的定义非常地“规范”，只能有public的抽象方法和常量组成。也就是说，接口中的方法是不能够在接口中直接实现的，只能在接口的实现类里来完成接口的实现。这样虽然最大限度地保证了接口的规范性，但是很多情况下降低了接口的可扩展性和便利性。比如在Java8中引入了lambda表达式的概念，我们要给Collection接口中添加一个forEach方法，如果接口不允许实现自己的方法，那就意味着Collection所有的实现类都需要添加forEach方法的实现，也就是说在Java8出现之前编写的所有Collection实现类如果不修改代码都会编译报错，即修改后的接口失去了向前兼容性。另外，如果接口中的某个方法在大多数情况下拥有相同的实现，那以在接口的每一个实现类中都需要编写相同的方法实现，这也就违背了DRY（don't repeat yourself）原则。

基于上述原因，在Java8中引入了一个新的概念——虚拟扩展方法，也就是通常说的defender方法，可以将其加入到接口中，这样可以提供方法的默认实现。实现类可以不重写默认方法，因此新添加的默认方法不会破坏接口的现有实现。因此可以这么说，接口的默认方法是lambdas表达式和JDK库之间的桥梁，它使得标准JDK接口得以进化。

再来聊一聊静态方法。过去版本的jdk中，为什么没有接口静态方法？关于这一点，可以参见[stackoverflow上的讨论](https://stackoverflow.com/questions/129267/why-no-static-methods-in-interfaces-but-static-fields-and-inner-classes-ok-pr/135722#135722)。根据官方文档的介绍，过去接口中没有静态方法并非强烈的技术原因导致的。设计师们本来提议在Java7中加入接口静态方法，但是由于不可预见的复杂性（unforeseen complications），将这一项设计推迟到了Java8中。接口静态方法可以改善JDK库的设计，例如Collections类中关于List操作的方法，本来可以放置在List接口中。

### API干货

接口的默认方法使用default关键字修饰，如：

```java
public interface I {
    default void method1(){
        System.out.println("调用I.method1()方法")
    }
    default void method2(){
        System.out.println("调用I.method2()方法")
    }
}

public class Clazz implements I {
    // 类中可以选择性重写接口的默认方法
    @Override
    public void method2(){
        System.out.println("调用Clazz.method2()方法")
    }
}
```

默认方法的设计有可能引发一定的问题，一个类在实现多个接口时，如果接口中有两个默认方法的方法签名相同，会发生什么情况？让我们先来看一下示例代码：

```java
public interface I {
    default void hello(){
        System.out.println("调用I.hello()方法")
    }   
}
public interface J {
    default void hello(){
        System.out.println("调用J.hello()方法")
    }   
}
public class Clazz implements I, J{
    
}
```

这时编译器分辨不出来到底应该继承哪个hello方法，于是编译器会报错：**inherits unrelated defaults for hello()from types I and J**。那么怎么解决这个问题呢？很简单，只要显式地重写hello方法即可：

```java
public class Clazz implements I, J{
    public void hello(){
        System.out.println("调用Clazz.hello()方法")
    }
}
```

还有更复杂的”菱形调用“，大家可以自行翻阅API。不刻意设计的情况下，一般是不会出现这种问题的。

静态方法的使用比较简单，和类中的静态方法没有什么区别：

```java
public interface A {  
    static void hello() {  
        System.out.println("调用A.hello()静态方法") 
    }  
}
public class Test {  
    public static void main(String[] args) {  
        A.hello();  
    }   
}
```

注意static和default关键字不能同时使用，即一个接口方法不可能既是静态的又是默认方法。

### 设计思路

几天前（2018年9月25日），Oracle发布了Java11。从Java8以来的几个版本的改动来看，Java的设计越来越务实，也越来越灵活。接口的默认方法和静态方法就是一个例子。从以前版本的观点来看，默认方法和静态方法是不太规范的东西。现在这些”不规范“特性的引入，也说明Java越来越包容，越来越迎合这个时代。默认方法的出现使得引入lambda表达式畅通无阻，使得Java最终加入了函数式编程的大军，大大简化了程序设计，提高了程序的可读性。接口静态方法的引入，使得接口设计的内聚性更强，避免了在设计中引入过多冗余的类。

### 最佳实践

编程不适合守旧者。既然有新东西，我们就应该果断学习和适应，果断拿来用。

利用接口默认方法我们可以来扩展旧有接口，也可以将通用的方法实现提炼到接口中。我们可以把原本适合设计成接口的抽象类重新设计为接口。

利用接口静态方法我们可以把属于接口中的静态方法还给接口，去掉外部冗余的方法和类。

### 练习与思考

1. 学习集合框架源码中接口default方法的使用。
2. 学习编写接口静态方法和默认方法的demo。