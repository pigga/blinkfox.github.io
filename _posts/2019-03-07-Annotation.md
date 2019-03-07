---
title: Annotation详解
date: 2019-03-07 11:06:00 +0800
layout: post
permalink: /blog/2019/03/07/Annotation.html
categories:
  - JAVA
tags:
  - Annotation
  - 基础语法
  - 注解
---

虽然已经见过多次Annotation,也亲身自使用过自定义注解.但对此还是不慎寥寥.特整理文档于此,备忘一二.
##### Annotation是什么
Annotation中文翻译为注解.啥玩意呢?其实看着也不太明白.<br/>
官方解释如下:
>Java注解用于为Java代码提供元数据。作为元数据，注解不直接影响你的代码执行，但也有一些
类型的注解实际上可以用于这一目的。Java注解是从Java5开始添加到Java 的。

懵吧,说实话看完之后还是各种似懂非懂.<br/>
看看百度百科怎么说的
>java.lang.annotation，接口 Annotation。对于Annotation，是Java5的新特性，JDK5引入了
Metadata（元数据）很容易的就能够调用Annotations。Annotations提供一些本来不属于程序的数
据，比如：一段代码的作者或者告诉编译器禁止一些特殊的错误。An annotation 对代码的执行没有
什么影响。Annotations使用@annotation的形式应用于代码：类(class),属性(attribute),方法
(method)等等。一个Annotation出现在上面提到的开始位置，而且一般只有一行，也可以包含有任意
的参数。

综合以上的描述,大概可以看到**Annotation其实说白了一般就是一行代码,可以包含任意参数.
Annotation能被用来为程序元素（类、方法、成员变量等）设置元数据。Annotaion不影响程序代码的执
行，无论增加、删除Annotation，代码都始终如一地执行。如果希望让程序中的Annotation起一定的作
用，只有通过解析工具或编译工具对Annotation中的信息进行解析和处理。**<br/>
_注意_:以上的描述中总有意无意的说一个词叫做元数据,那么什么是元数据呢?
###### 元数据是什么
>元数据（Metadata）,又称中介数据、中继数据,为描述数据的数据（data about data）,主要是描
述数据属性（property）的信息,用来支持如指示存储位置、历史数据、资源查找、文件记录等功能.

示例如下：<br/>
1）编写文档：通过代码里标识的元数据生成文档<br/>
2）代码分析：通过代码里标识的元数据对代码进行分析<br/>
3）编译检查：通过代码里标识的元数据让编译器能实现基本的编译检查<br/>

**Java平台元数据**
注解Annotation就是java平台的元数据，是 J2SE5.0新增加的功能，该机制允许在Java 代码中添加自
定义注释，并允许通过反射（reflection），以编程方式访问元数据注释。通过提供为程序元素（类、方
法等）附加额外数据的标准方法，元数据功能具有简化和改进许多应用程序开发领域的潜在能力，其中包括
配置管理、框架实现和代码生成。

##### Annotation怎么用
简单的了解之后,我们先简单的写一个注解.
```
public @interface TestAnnotation {

}
```
如上所示,即使一个简单的注解.使用的话就直接在想要使用的类、方法、成员变量上直接使用即可.
```
@TestAnnotation
public class Demo {

}
```
注解通过`@interface`关键字进行定义,看着与接口的唯一区别是interface前多了个@符号.不过上面的
示例中的注解没有什么内容,无法有效的达到什么目的.那么为了具体使用它,我们必须先比较一下元注解的
概念.
###### 元注解是什么
元注解是可以注解到注解上的注解，或者说元注解是一种基本注解，但是它能够应用到其它的注解上面.元注
解有@Retention、@Documented、@Target、@Inherited、@Repeatable5种。
- @Retention
Retention 的英文意为保留期的意思。当@Retention 应用到一个注解上的时候，它解释说明了这个注解
的的存活时间。

它的取值如下：
```
RetentionPolicy.SOURCE 注解只在源码阶段保留,在编译器进行编译时它将被丢弃忽视。
RetentionPolicy.CLASS 注解只被保留到编译进行的时候,它不会被加载到JVM中。
RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候,它会被加载进入到 JVM中，在程序运行时可以获取到它们。
```
具体使用如下
```
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {

}
```
说明`TestAnnotation`注解可以在程序运行时,获取到
- @Documented
一个简单的Annotations标记注解，表示是否将注解信息添加在java文档中。
- @Target
Target中文意思是目标,靶子.当使用了`@Target`时,说明对应的修饰注解是有指定目标的.<br/>
具体的取值范围如下:
```
ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
ElementType.CONSTRUCTOR     可以给构造方法进行注解
ElementType.FIELD           可以给属性进行注解
ElementType.LOCAL_VARIABLE  可以给局部变量进行注解
ElementType.METHOD          可以给方法进行注解
ElementType.PACKAGE         可以给一个包进行注解
ElementType.PARAMETER       可以给一个方法内的参数进行注解
ElementType.TYPE            可以给一个类型进行注解，比如类、接口、枚举
```
一旦加有`@Traget`的标签被使用到了错误位置,会在编译期直接报错.
- @Inherited
`@Inherited`指定被它修饰的Annotation将具有继承性——如果某个类使用了@Xxx注解（定义该
Annotation时使用了@Inherited修饰）修饰，则其子类将自动被@Xxx修饰。
- @Repeatable
Repeatable 自然是可重复的意思。@Repeatable 是 Java 1.8 才加进来的，所以算是一个新的特性。<br/>
什么样的注解会多次应用呢？通常是注解的值可以同时取多个。<br/>
举个例子，一个人他既是程序员又是产品经理,同时他还是个画家.<br/>
```
@interface Persons {
	Person[] value();
}

@Repeatable(Persons.class)
@interface Person{
	String role default "";
}

@Person(role="artist")
@Person(role="coder")
@Person(role="PM")
public class SuperMan{
	
}
```

看着感觉很怪,也难以理解.先顺着往下看.
##### 注解的属性
注解的属性也叫做成员变量。注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无形参的方
法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型。

```
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {

    int id();
    String name();
}
```
上面的代码定义了`TestAnnotation`注解下的两个属性,`id`与`name`.<br/>
但是在使用的时候也要同步的在注解内为对应的属性赋上值.
```
@TestAnnotation(id = 2, name = "yannis")
public class Demo {

    public void get_answer(){
    }
}
```
**注意**，在注解中定义属性时它的类型必须是8种基本数据类型外加类、接口、注解及它们的数组。<br/>
注解的属性可以有默认值,默认值使用default指定.
```
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {

    int id() default 1;
    String name() default "yannis";
}
```
若是注解内的属性已经指明了默认值,则使用他们的时候可以不需要在注解上为对应的属性赋值.
```
@TestAnnotation
```
```
@TestAnnotation()
```
```
@TestAnnotation(id = 2, name = "yannis")
```
如上三种皆可以.<br/>
_特殊情况_,若注解内仅有一个属性`value`,则在使用的时候,可以不指定属性名.
```
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {

    int value();
}

@TestAnnotation(2)
public class Demo {

}
```
##### 注解的原理
>注解本质是一个继承了 Annotation 的特殊接口，其具体实现类是 Java 运行时生成的动态代理类。而
我们通过反射获取注解时，返回的是 Java 运行时生成的动态代理对象 $Proxy1。通过代理对象调用自定
义注解（接口）的方法，会最终调用 AnnotationInvocationHandler 的 invoke 方法。该方法会从
memberValues 这个 Map 中索引出对应的值。而 memberValues 的来源是 Java 常量池。
——摘自《注解Annotation实现原理与自定义注解例子》
##### 注解的使用
注解在创建完之后,永远不会主动干预代码行为,所以为了更好的利用注解,需要使用反射,获取相关注解.
首先可以通过 Class 对象的 isAnnotationPresent() 方法判断它是否应用了某个注解
```
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {}
```
然后通过 getAnnotation() 方法来获取 Annotation 对象。
```
public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {}
```
或者是 getAnnotations() 方法。
```
public Annotation[] getAnnotations() {}
```
前一种方法返回指定类型的注解，后一种方法返回注解到这个元素上的所有注解。

如果获取到的 Annotation 如果不为 null，则就可以调用它们的属性方法了。比如
```
@TestAnnotation
public class Demo {

    public static void main(String[] args) {
        boolean hasAnnotationFlag = Demo.class.isAnnotationPresent(TestAnnotation.class);
        if(hasAnnotationFlag){
            TestAnnotation testAnnotation = Demo.class.getAnnotation(TestAnnotation.class);
            System.out.println(testAnnotation.id());
            System.out.println(testAnnotation.name());
        }
    }
}
```
至此关于注解的说明告一段落,后续若有其他的,再补充.
参考: https://blog.csdn.net/briblue/article/details/73824058
https://www.jianshu.com/p/4b79ce0b628c
https://www.jianshu.com/p/f90504e1d127