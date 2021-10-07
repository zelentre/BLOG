---
title: Java学习——异常类
date: 2021-10-06 16:22:50
categories: 
 - Java
tags: 
 - 异常类
---

# Java学习——异常类

## 一、什么是异常

​	异常就是在运行时产生的问题。通常用<font color='red'>Exception</font>描述。在Java中，把异常<font color='red'>封装成了一个类</font>，当出现问题时，就会创建异常类对象并抛出异常相关的信息（如详细信息，名称以及异常所处的位置）

我们写 Java 程序经常会出现两种问题，一种是`java.lang.Exception` ，一种是 `java.lang.Error`，都用来表示出现了异常情况，下面就针对这两种概念进行理解。

<!--more-->

## 二、什么是 Throwable

Throwable 类是 Java 语言中所有`错误(errors)`和`异常(exceptions)`的父类。只有继承于 Throwable 的类或者其子类才能够被抛出，还有一种方式是带有 Java 中的 `@throw` 注解的类也可以抛出。

> Throwable: 它是所有错误与异常的超类（父类）
>     |- Error 错误
>     |- Exception 编译期异常,进行编译JAVA程序时出现的问题
>         |- RuntimeException 运行期异常, Java程序运行过程中出现的问题

在[Java规范](https://docs.oracle.com/javase/specs/jls/se9/html/jls-11.html#jls-11.1.1)中，对非受查异常和受查异常的定义是这样的：

> The *unchecked exception classes* are the run-time exception classes and the error classes.

> The *checked exception classes* are all exception classes other than the unchecked exception classes. That is, the checked exception classes are `Throwable` and all its subclasses other than `RuntimeException` and its subclasses and `Error`and its subclasses.

也就是说，除了 `RuntimeException` 和其子类，以及`error`和其子类，其它的所有异常都是 `checkedException`。

那么，按照这种逻辑关系，我们可以对 Throwable 及其子类进行归类分析

![img](https://gitee.com/zelen/IMG/raw/master/PicGo/68747470733a2f2f696d67323032302e636e626c6f67732e636f6d2f626c6f672f313531353131312f3230323030342f313531353131312d32303230303430383134303330313935362d313936303630373134362e706e67)

可以看到，Throwable 位于异常和错误的最顶层，我们查看 Throwable 类中发现它的方法和属性有很多，我们只讨论其中几个比较常用的

```java
// 返回抛出异常的详细信息
public string getMessage();
public string getLocalizedMessage();

// 返回异常发生时的简要描述
public public String toString()；
  
// 打印异常信息到标准输出流上
public void printStackTrace();
public void printStackTrace(PrintStream s);
public void printStackTrace(PrintWriter s)

// 记录栈帧的的当前状态
public synchronized Throwable fillInStackTrace();
```

此外，因为 Throwable 的父类也是 `Object`，所以常用的方法还有继承其父类的`getClass()` 和 `getName()` 方法。

## 三、什么是Exception

​	`Exception` 位于 `java.lang` 包下，它是一种顶级接口，继承于 `Throwable` 类，Exception 类及其子类都是 Throwable 的组成条件，用来表示Java中可能出现的异常，并且合理的处理这些异常。

​	Exception 有两种异常，一种是 `RuntimeException` ；一种是 `CheckedException`，这两种异常都应该去`捕获`。

​	**RuntimeException**

| 异常名称                       | 异常描述         |
| ------------------------------ | ---------------- |
| ArrayIndexOutOfBoundsException | 数组越界异常     |
| NullPointerException           | 空指针异常       |
| IllegalArgumentException       | 非法参数异常     |
| NegativeArraySizeException     | 数组长度为负异常 |
| IllegalStateException          | 非法状态异常     |
| ClassCastException             | 类型转换异常     |

​	**UncheckedException**

| 异常名称               | 异常描述                         |
| ---------------------- | -------------------------------- |
| NoSuchFieldException   | 表示该类没有指定名称抛出来的异常 |
| NoSuchMethodException  | 表示该类没有指定方法抛出来的异常 |
| IllegalAccessException | 不允许访问某个类的异常           |
| ClassNotFoundException | 类没有找到抛出异常               |

### 与 Exception 有关的 Java 关键字

那么 Java 中是如何处理这些异常的呢？在 Java 中有这几个关键字 **throws、throw、try、finally、catch** 下面我们分别来探讨一下

#### throws 和 throw

​	在Java中，异常也就是一个对象，它能够被程序员自定义抛出或者应用程序抛出，必须借助于`throws`和`throw`语句来定义抛出异常。

throws和throw通常是成对出现的，例如

```java
static void cacheException() throws Exception{
    throw new Exception();
}
```

throw语句用在方法体内，表示抛出异常，由方法体内的语句处理。throws语句用在方法声明后，表示再抛出异常，由该方法的调用者来处理。

throws主要是声明这个方法会抛出这种类型的异常，使它的调用者知道要捕获这个异常。throw是具体向外抛出异常的动作，所以它抛出的是一个异常实例

#### try、finally、catch

这三个关键字主要有下面几种组合方式 **try...catch、try...finally、try...catch...finally**

try...catch表示对某一段代码可能抛出的异常进行的捕获，如下

```java
static void cacheException() throws Exception{
    try{
      System.out.println("1");  
    }catch(Exception e){
      e.printStackTrace();
    }
}
```

try...finally表示对一段代码不管执行情况如何，都会走finally中的代码

```java
static void cacheException() throws Exception{
  for (int i = 0; i < 5; i++) {
    System.out.println("enter: i=" + i);
    try {
      System.out.println("execute: i=" + i);
      continue;
    } finally {
      System.out.println("leave: i=" + i);
    }
  }
}
```

try...catch...finally 也是一样的，表示对异常捕获后，再走 finally 中的代码逻辑。

#### JDK1.7 使用 try...with...resources 优雅关闭资源

要使用 try-with-resources语句，首先要实现`AutoCloseable`接口，此接口包含了单个返回的close方法。Java类库与第三方类库中的许多类和接口，现在都实现了`AutoCloseable`接口。如果编写了一个类，它代表的是必须关闭的资源，那么这个类应该实现`AutoCloseable`接口。

Java引入了 try-with-resources声明，将 try-catch-finally简化为try-catch，这其实是一种**语法糖**，在编译时会进行转化，转化为try-catch-finally语句。

> 使用 try-with-resources不仅使代码变得通俗易懂也更容易诊断。以`firstLineOfFileAutoClose`方法为例，如果调用`readLine()`和`close()`方法都抛出异常，后一个异常就会被禁止，以保留第一个异常。

### 异常处理的原则

- 不要捕获类似`Exception`之类的异常，而应该捕获类似特定的异常，比如`InterruptedException`，方便排查问题
- 不要生吞异常。这是异常处理中要特别注重的事情，因为很可能会非常难以正常结束情况。如果我们不把异常抛出来，或者也没有输出到Logger日志中，程序可能会在后面以不可控的方式结束
- 不要在函数式编程中使用`checkedException`

## 四、什么是Error

Error是程序无法处理的错误，表示运行应用程序中较严重的问题。大多数错误和代码编写者执行的操作无关，而表示代码运行时JVM（Java虚拟机）出现的问题。这些错误时不可检查的，因为它们在应用程序的控制和处理能力之外，而且绝大多数是程序运行时不允许出现的状况，比如`OutOfMemoryError`和 `StackOverflowError`异常的出现会有几种情况

## 相关参考

1. [这是一个成为更好的Java程序员的系列教程 (github.com)](https://github.com/crisxuan/bestJavaer)
2. [Java学习（异常类）](https://www.cnblogs.com/0328dongbin/p/9186676.html)
