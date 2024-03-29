---
title: Spring事务
date: 2022-10-14 16:47:54
categories: 
 - spring
tags: 
 -  spring事务
---

> 事务其实是一个并发控制单位，是用户定义的一个操作序列，这些操作要么全部完成，要不全部不完成，是一个不可分割的工作单位。事务有ACID四个特性，即：
>
> 1. Atomicity（原子性）：事务中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。
> 2. 一致性（Consistency）：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。
> 3. 事务隔离（Isolation）：多个事务之间是独立的，不互相影响的。
> 4. 持久性（Durability）：事务处理结束后，对数据的修改就是永久的，即使系统故障也不会丢失。

<!--more-->

> MySQL默认情况下，对于所有的单条语句都作为一个单独的事务来执行。我们要使用MySQL事务的时候，可以通过手动提交事务来控制事务范围。**Spring事务的本质，其实就是通过 Spring AOP 切面技术，在合适的地方开启事务，接着在合适的地方提交事务或回滚事务，从而实现了业务编程层面的事务操作。**

## 事务传播类型

事务传播类型，指的是事务与事务之间的交互策略。Spring事务中定义了7种事务传播类型，分别是：REQUIRED、SUPPORTS、MANDATORY、REQUIRES_NEW、NOT_SUPPORTED、NEVER、NESTED。其中最常用的只有 3 种，即：REQUIRED、REQUIRES_NEW、NESTED。

针对事务传播类型，我们要弄明白的是 4 个点：

1. 子事务与父事务的关系，是否会启动一个新的事务？
2. 子事务异常时，父事务是否会回滚？
3. 父事务异常时，子事务是否会回滚？
4. 父事务捕捉异常后，父事务是否还会回滚？

 ### REQUIRED

> REQUIRED是Spring默认的事务传播类型，该传播类型的特点是：**当前方法存在事务时，子方法加入该事务。此时父子方法共用一个事务，无论父子方法那个发生异常回滚，整个事务都回滚。即使父方法捕获了异常也是会回滚。而当前方法不存在事务时，子方法新建一个事务。**

### REQUIRES_NEW

> REQUIRES_NEW也是常用的一个事务传播类型，该类型的特点是：**无论当前方法是否存在事务，子事务都会新建一个事务。此时父子方法的事务是独立的，它们都不会互相影响。但父方法需要注意子方法抛出的异常，避免因子方法抛出异常，而导致的父方法回滚。**

### NESTED

>  该方法的特性与REQUIRED非常相似，其特性是：**当前方法存在事务时，子方法加入在嵌套事务执行。当父方法事务回滚时，子方法事务也跟着回滚。当子方法事务发生回滚时，父方法事务是否回滚取决于是否捕获了异常。如果捕获了异常，那么就不会滚，否则回滚。**

总结：

| 事务传播类型 | 特性                                                         |
| ------------ | ------------------------------------------------------------ |
| REQUIRED     | 当前方法存在事务时，子方法加入该事务。此时父子方法共用一个事务，无论父子方法哪个发生异常回滚，整个事务都回滚。即使父方法捕捉了异常，也是会回滚。而当前方法不存在事务时，子方法新建一个事务。 |
| REQUIRES_NEW | 无论当前方法是否存在事务，子方法都新建一个事务。此时父子方法的事务时独立的，它们都不会相互影响。但父方法需要注意子方法抛出的异常，避免因子方法抛出异常，而导致父方法回滚。 |
| NESTED       | 当前方法存在事务时，子方法加入在嵌套事务执行。当父方法事务回滚时，子方法事务也跟着回滚。当子方法事务发送回滚时，父事务是否回滚取决于是否捕捉了异常。如果捕捉了异常，那么就不回滚，否则回滚。 |

## 事务什么情况会失效

1. 内部调用
   - 使用一个没有事务的方法调用一个有事务的方法，失败后不会进行回滚
2. 没有指定监听的Exception
   - 如果抛出非 RuntimeException 和非 Error 错误的其他异常，不能正常捕获，也就不会回滚。
3. 内部异常被catch

4. 方法非public

##  相关参考

[深入理解 Spring 事务：入门、使用、原理](https://mp.weixin.qq.com/s/7f7VvAK-A3ab2HSCATm-Fw)

[springboot事务什么情况会失效](https://blog.csdn.net/qq_22795957/article/details/108157179)

[springboot事务什么情况会失效](https://mp.weixin.qq.com/s/FVPe4OrC33px02ldqzkTaQ)
