---
title: Interview
date: 2021-09-07 11:37:24
categories: 
 - interview
tags: 
 - interview
---

# Interview

<!-- more -->

## 问题

### 一、Controller默认是单例还是多例，单例会出现什么问题，有成员变量怎么解决？相关参考 1

> Controller默认是单例的，不要使用非静态的成员变量，否则会发生数据逻辑混乱。正因为单例所以不是线程安全的。
>
> **单例是不安全的，会导致属性重复使用。**
>
> **解决方案**
>
> 1. 不要在controller中定义成员变量
> 2. 万一必须要定义一个非静态成员变量时候，则通过注解@Scope(“prototype”)，将其设置为多例模式
> 3. 在Controller中使用ThreadLocal变量

## 相关参考

1. [Controller 是单例还是多例？怎么保证并发的安全](https://cxyroad.blog.csdn.net/article/details/113903802)、[springmvc控制器controller单例问题](https://www.cnblogs.com/eric-fang/p/5629892.html)

