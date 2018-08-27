---
layout: post
title:  "提高开发效率的奇技淫巧（一）lombok"
categories: Java
tags:  lombok Effeciency
author: G. Seinfeld
---

* content
{:toc}

### 闲话
“程序猿”、“码农”、“软件攻城狮”，程序员这个职业现在已经被这些网络流行语给玩坏了。由于程序员门槛越来越低，由于“copy+改”的开发模式在业界里的流行，“程序员”这个曾经神圣的词语也似乎越来越廉价了。在人们的印象中，程序员似乎就是整天在噼哩啪啦敲键盘的职业。我想说，噼哩啪啦敲键盘的那群人，其实叫打字员。程序员起码应该是一个动脑子的打字员。一个典型的程序员（或者叫合格的程序员），应该至少有80%的时间在思考和学习，最多有20%的时间用于码字。也就是说，一个不加班的程序员（假设存在），一天应该最多有1.6个小时在敲代码。1.6小时能写多少代码？对于Java程序员，有效的代码量应该差不多是100行。Java恰恰是一个无效代码很多的语言，Java哆嗦的表达方式被无数业内人数所诟病。因此，如何降低无效代码量，提高开发效率，是值得每个Java程序员应该思考的问题。

开这一个系列旨在帮助大家提高开发效率，主要是介绍一些第三方包、插件等的使用。在开始之前，这里有几个优先级更高的通用性建议：

一是选择一款合适的IDE（Integrate Development Environment，集成开发环境）。Java本身的繁琐使得它不适合使用记事本类轻型开发工具进行开发，因此推荐使用相对重量级的IDE进行开发。合适的开发工具可以帮助我们完成很多重复性工作，如生成常用的代码、自动导包、自动整理代码格式，大幅节省码字时间。目前市面上流行的Java IDE主要有三款：eclipse（包括在此基础上衍生的MyEclipse、STS等）、intellij idea、netbeans。这三款产品各有所长，但综合比较下来，强烈推荐使用intellij idea，它的智能提示效果和对各种文件类型的支持是其他两款IDE所难以启及的。顺便一提，它是捷克的一家叫做JetBrains的公司出品的IDE，这家公司出品的其他语言的IDE也很优秀，如用于写python的PyCharm、写php的PhpStorm、写前端的WebStorm等。当然，idea的专业版是收费的，大家都懂的，有条件的同学请支持正版。

二是建立一个自己的常用代码库，将一些新项目中常使用的代码保存进去，随取随用，可以使用github的gist功能帮助自己维护这个代码库，idea有直接向github上提交gist的快捷功能，大家可以自行研究一下，后续文章中会有详细介绍。

三是使用新版本的JDK。在当前语境下，建议使用JDK8以上的版本。Java8的lambda表达式和Stream功能可以大幅提高开发效率。每个版本的JDK设计时，都会考虑简化开发、提高效率，比如java7的diamond语法（Map<String, Object> map = new HashMap<>()，等号后面的尖括号内不用再指定类型），再比如后续java10中的var使得java向动态类型语言的方向发展。因此，掌握和使用新版本的JDK几乎总能提升你的开发效率。

四是学习一门脚本语言，比如python。一方面平时常用的一些小的功能可以用脚本语言快速开发出来，另一方面可以使用脚本语言结合模板开发一些代码生成器。有人说过，超过90秒的重复性工作就应该写脚本来完成，试想你的代码能代替你工作，你是不就可以坐享其成了？

这个系列的第一部分讲lombok，它是一个旨在减少重复性代码的第三方包，它的设计思路是通过一系列注解来自动生成代码。

### 引入方式
在maven的pom.xml文件中添加lombok的坐标，如：
```xml
    <dependency> 
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.16.18</version>
    </dependency>
```
注意在idea中使用lombok注解需要安装lombok插件，如下图：
![idea_lombok_plugin](resources/idea_lombok_plugin.png)

### API干货

#### Data、Value
这两个注解是“一站式”的注解，设计的目的是想要取代一个实体类中除成员变量声明以外的其他所有代码。这两个注解都用在类的上面，二者的区别在于@Data注解是按照可变类的方式生成代码，@Value注解是按照不可变类的方式生成代码。按照文档上的说法，@Data注解相当于@Getter @Setter @RequiredArgsConstructor @ToString @EqualsAndHashCode注解的“五合一”合集，它会根据成员变量自动生成相应的get方法、set方法、构造器、toString方法以及equals和hashCode方法。注意final修饰的成员变量不会生成相应的set方法，也不会参与构造器的生成，transient修饰的成员变量则不会参与equals和hashCode方法的生成。@Data注解有一个可选项staticConstructor，可以通过将该选项的值设置为of来生成一个静态的构造器。@Value注解相当于@Getter @FieldDefaults(makeFinal=true, level=AccessLevel.PRIVATE) @AllArgsConstructor @ToString @EqualsAndHashCode的“五合一”合集，它会按照不可变类的方式去生成代码。它会将所有成员变量声明为private final变量，生成包含所有变量的构造器以及getter、toString、equals和hashCode方法。下面看一下实际效果，假设有一个Student类，包含id、name、age、grade字段：
```java
@Data
public class Student{
    private String id;
    private final String name;
    private transient int age;
    private int grade;
}
```
等价于
```java
import java.beans.ConstructorProperties;

public class Student {
    private String id;
    private final String name;
    private transient int age;
    private int grade;

    @ConstructorProperties({"name"})
    public Student(String name) {
        this.name = name;
    }

    public String getId() {
        return this.id;
    }

    public String getName() {
        return this.name;
    }

    public int getAge() {
        return this.age;
    }

    public int getGrade() {
        return this.grade;
    }

    public void setId(String id) {
        this.id = id;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setGrade(int grade) {
        this.grade = grade;
    }

    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof Student)) {
            return false;
        } else {
            Student other = (Student)o;
            if (!other.canEqual(this)) {
                return false;
            } else {
                label39: {
                    Object this$id = this.getId();
                    Object other$id = other.getId();
                    if (this$id == null) {
                        if (other$id == null) {
                            break label39;
                        }
                    } else if (this$id.equals(other$id)) {
                        break label39;
                    }

                    return false;
                }

                Object this$name = this.getName();
                Object other$name = other.getName();
                if (this$name == null) {
                    if (other$name != null) {
                        return false;
                    }
                } else if (!this$name.equals(other$name)) {
                    return false;
                }

                if (this.getGrade() != other.getGrade()) {
                    return false;
                } else {
                    return true;
                }
            }
        }
    }

    protected boolean canEqual(Object other) {
        return other instanceof Student;
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $id = this.getId();
        int result = result * 59 + ($id == null ? 43 : $id.hashCode());
        Object $name = this.getName();
        result = result * 59 + ($name == null ? 43 : $name.hashCode());
        result = result * 59 + this.getGrade();
        return result;
    }

    public String toString() {
        return "Student(id=" + this.getId() + ", name=" + this.getName() + ", age=" + this.getAge() + ", grade=" + this.getGrade() + ")";
    }
}
```
```java
@Value
public class Student {
    private String id;
    private String name;
    private transient int age;
    private int grade;
}
```
则等价于
```java
import java.beans.ConstructorProperties;

public final class Student {
    private final String id;
    private final String name;
    private final transient int age;
    private final int grade;

    @ConstructorProperties({"id", "name", "age", "grade"})
    public Student(String id, String name, int age, int grade) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.grade = grade;
    }

    public String getId() {
        return this.id;
    }

    public String getName() {
        return this.name;
    }

    public int getAge() {
        return this.age;
    }

    public int getGrade() {
        return this.grade;
    }

    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof Student)) {
            return false;
        } else {
            Student other = (Student)o;
            Object this$id = this.getId();
            Object other$id = other.getId();
            if (this$id == null) {
                if (other$id != null) {
                    return false;
                }
            } else if (!this$id.equals(other$id)) {
                return false;
            }

            label29: {
                Object this$name = this.getName();
                Object other$name = other.getName();
                if (this$name == null) {
                    if (other$name == null) {
                        break label29;
                    }
                } else if (this$name.equals(other$name)) {
                    break label29;
                }

                return false;
            }

            if (this.getGrade() != other.getGrade()) {
                return false;
            } else {
                return true;
            }
        }
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $id = this.getId();
        int result = result * 59 + ($id == null ? 43 : $id.hashCode());
        Object $name = this.getName();
        result = result * 59 + ($name == null ? 43 : $name.hashCode());
        result = result * 59 + this.getGrade();
        return result;
    }

    public String toString() {
        return "Student(id=" + this.getId() + ", name=" + this.getName() + ", age=" + this.getAge() + ", grade=" + this.getGrade() + ")";
    }
}

```
注意使用@Value注解时，成员变量不能直接声明为final类型，否则会有编译错误。

#### Builder
@Builder用于实现23种设计模式中的构建器模式，该模式通常用于构造包含多个成员变量的类。如果一个类拥有多个成员变量，创造包含全部成员变量的构造器不够灵活，都使用setter方法赋值又过于麻烦，这时构建器模式就发挥作用了。构建器模式声明一个公有的静态构建器，然后声明所有成员变量的同名方法，返回构建器，这样就可以实现链式调用，最后通过build()方法调用私有构造器，完成对象的创建，从而简化代码。下面看一下官方文档里的例子：
```java
@Builder
class Example {
	private int foo;
	private final String bar;
}
```
等价于
```java
class Example {
    private int foo;
    private final String bar;

    Example(int foo, String bar) {
        this.foo = foo;
        this.bar = bar;
    }

    public static Example.ExampleBuilder builder() {
        return new Example.ExampleBuilder();
    }

    public static class ExampleBuilder {
        private int foo;
        private String bar;

        ExampleBuilder() {
        }

        public Example.ExampleBuilder foo(int foo) {
            this.foo = foo;
            return this;
        }

        public Example.ExampleBuilder bar(String bar) {
            this.bar = bar;
            return this;
        }

        public Example build() {
            return new Example(this.foo, this.bar);
        }

        public String toString() {
            return "Example.ExampleBuilder(foo=" + this.foo + ", bar=" + this.bar + ")";
        }
    }
}
```
@Builder注解可以用于类、方法和构造器上，如果@Builder注解作用于方法上，生成的build()方法会调用该方法，不可以有两个方法同时使用@Builder注解。

#### Getter、Setter、ToString、EqualsAndHashCode
@Getter、@Setter、@ToString和@EqualsAndHashCode注解分别用于给类的成员变量生成get方法、set方法、给类生成toString()、equals和hashCode()方法。
其中@Getter和@Setter注解可以作用于类上或者成员变量上，如果作用于类上，所有的成员变量均自动生成get和set方法（final变量无set方法）。注意如果变量是boolean类型，生成的get方法叫做isXXX，如boolean good的get方法为isGood()。如果想生成非public的get或set方法，可以将注解的value属性进行设置，如：@Getter(value=lombok.AccessLevel.PROTECTED)
@ToString注解用于生成类的toString()方法，仅能作用于类上。可用的属性值包括of、exclude、includeFieldNames以及callSuper。of用于指定包含哪些字段，exclude用于指定排除哪些字段，of和exclude只能存在一个。includeFieldNames是一个boolean值，默认值为true，用于指定是否包含成员变量名，callSuper也是一个boolean值，默认值为false，如果设置为true，在生成的toString方法中将包含父类的toString结果。
@EqualsAndHashCode方法用于生成类的equals()和hashCode()方法，只能作用于类上，可用的属性值类似于@ToString，也包括of、exclude以及callSuper。transient修饰的成员变量不参与生成equals()和hashCode()方法。
下面看一些例子：
```java

@ToString(includeFieldNames=false, of={"id", "grade"})
@EqualsAndHashCode(exclude={"name"})
public class Student {
    @Getter(value=lombok.AccessLevel.PROTECTED)
    private String id;
    @Setter
    private String name;
    private transient int age;
    private int grade;
}
```
等价于
```java
public class Student {
    private String id;
    private String name;
    private transient int age;
    private int grade;

    public Student() {
    }

    public String toString() {
        return "Student(" + this.getId() + ", " + this.grade + ")";
    }

    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof Student)) {
            return false;
        } else {
            Student other = (Student)o;
            if (!other.canEqual(this)) {
                return false;
            } else {
                Object this$id = this.getId();
                Object other$id = other.getId();
                if (this$id == null) {
                    if (other$id == null) {
                        return this.grade == other.grade;
                    }
                } else if (this$id.equals(other$id)) {
                    return this.grade == other.grade;
                }

                return false;
            }
        }
    }

    protected boolean canEqual(Object other) {
        return other instanceof Student;
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $id = this.getId();
        int result = result * 59 + ($id == null ? 43 : $id.hashCode());
        result = result * 59 + this.grade;
        return result;
    }

    protected String getId() {
        return this.id;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

#### NoArgsConstructor、RequiredArgsConstructor、AllArgsConstructor
@NoArgsConstructor、@RequiredArgsConstructor和@AllArgsConstructor都作用于类上，分别用于生成无参构造器、指定参数构造器和全参构造器。@RequiredArgsConstructor指定的参数是指final修饰的成员变量以及有约束条件（如用@NonNull修饰）的成员变量。

#### Cleanup
@Cleanup注解用在局部变量上，用于关闭指定的资源，如输入输出流。@Cleanup注解的底层实现方式是在finally块中调用资源的close方法，如果关闭资源的方法名不为close，可以使用注解的value属性指定方法名，注意指定的方法必须是没有参数的。看下官方文档中的例子：
```java
public void copyFile(String in, String out) throws IOException {
    @Cleanup FileInputStream inStream = new FileInputStream(in);
    @Cleanup FileOutputStream outStream = new FileOutputStream(out);
    byte[] b = new byte[65536];
    while (true) {
        int r = inStream.read(b);
        if (r == -1) break;
        outStream.write(b, 0, r);
    }
}
```
等价于
```java
public void copyFile(String in, String out) throws IOException {
   @Cleanup FileInputStream inStream = new FileInputStream(in);
   try {
       @Cleanup FileOutputStream outStream = new FileOutputStream(out);
       try {
           byte[] b = new byte[65536];
           while (true) {
               int r = inStream.read(b);
               if (r == -1) break;
               outStream.write(b, 0, r);
           }
       } finally {
           if (out != null) out.close();
       }
   } finally {
       if (in != null) in.close();
   }

```

#### NonNull
@NonNull注解作用于类的成员变量、方法、参数以及局部变量上。如果放在参数上，lombok将在方法/构造器方法体内最开始的位置插入空值检测的语句，如果变量值为null，将抛出空指针异常。如果放在成员变量上，任何为该变量赋值的方法（如set方法和构造器）中将生成空值检测语句。在方法和局部变量上加入该注解似乎没有什么作用。

#### SneakyThrows
@SneakyThrows注解作用于方法或构造器上，用于将受检异常转换为非受检异常，该方法可能导致程序暗藏杀机，不推荐使用。

#### Synchronized
@Synchronized注解作用于方法上，类似于synchronized关键字，它生成一个私有的变量，将同步锁加在私有变量上，这样可以避免其他不受你控制的代码干扰你的线程管理。借用一个别人整理的代码：
```java
public class SynchronizedExample {
  private final Object readLock = new Object();
  
  @Synchronized
  public static void hello() {
    System.out.println("world");
  }
  
  @Synchronized
  public int answerToLife() {
    return 42;
  }
  
  @Synchronized("readLock")
  public void foo() {
    System.out.println("bar");
  }
}
```
等价于
```java
public class SynchronizedExample {
  private static final Object $LOCK = new Object[0];
  private final Object $lock = new Object[0];
  private final Object readLock = new Object();
  
  public static void hello() {
    synchronized($LOCK) {
      System.out.println("world");
    }
  }
  
  public int answerToLife() {
    synchronized($lock) {
      return 42;
    }
  }
  
  public void foo() {
    synchronized(readLock) {
      System.out.println("bar");
    }
  }
}

```

#### val
@val应该是所有注解里最特殊的一个了。首先它长得就不太一样，首字母小写让它看上去不那么像注解。再者它的用法也不太一样，它使用的时候不需要加“@”符号，直接
```java
val s = "code";
```
即可。这个注解用于局部变量声明时自动推断变量的类型，如上述代码将s推断为String类型。它是一项“未来的”java特性，是对java10中var关键字的一种补充。val关键字声明的局部变量都是不可变的，var关键字修饰的变量则为可变的。
注意本质上val还是一个注解，即
```java
val x = 10;
```
相当于
```java
@val final int x = 10;
```

### 设计思路
有的童鞋可能会问：你上面说的这些玩意，IDE基本都可以自动生成啊，lombok到底有意义么？
下面几点可以证明简化的比生成的好：
- 可读性。lombok去掉了getter、setter、toString、equals、hashCode等一系列冗长的代码，使得实体类看上去更加清爽。
- 便于修改。虽然IDE可以自动生成getter、setter和构造器，但是对于修改来说简直是噩梦。比如你要把一个字段从String改成int类型，你需要把对应的getter、setter、构造器、toString、equals、hashCode通通删掉，再重新生成。使用lombok的话这些工作都不用做了，直接修改就可以了。

### 最佳实践
总结一下，lombok主要用于简化Java重复代码，提高开发效率：
- @Data、@Value这两个合集用于简化实体类的重复代码，一个可变一个不可变，在一般情况下这俩注解就够用了。但是如果要订制getter、setter等方法，就需要使用各个注解了。
- @Builder注解用于实现构造器模式，方便多成员变量的实体类的构造。
- @Cleanup注解用于关闭资源，功能类似于try-with-resources特性。
- @NonNull注解用于对参数和成员变量进行空值检测。
- @Synchronized注解用于更优雅地实现同步锁。
- @val注解用于体验java未来的功能——动态变量类型。
