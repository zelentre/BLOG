---
title: synchronized锁升级之无锁和偏向锁
date: 2022-11-07 21:43:44
categories: 
 - java
tags: 
 - synchronized
---

# 一、无锁

> 为了优化`synchronized`锁的效率，在`JDK6`中，`HotSpot`虚拟机开发团队提出了锁升级的概念，包括偏向锁、轻量级锁、重量级锁等，锁升级指的就是“**无锁 --> 偏向锁 --> 轻量级锁 --> 重量级锁**”。

synchronized同步锁相关信息保存到锁对象的对象头里面的Mark Word中，锁升级功能主要是依赖Mark Word中锁标志位和是否偏向锁标志位来实现的。

![image-20221127202208302](https://testingcf.jsdelivr.net/gh/znej/pic/picgo/image-20221127202208302.png)

从上图我们可以看到，无锁对应的锁标志位是`01`,是否偏向锁标志是`0`
