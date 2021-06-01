---
title: JVM相关
date:  2021-04-23 15:29:16  
categories: 
 - Java
 - jvm
tags: 
 - Java基础
 - jvm
---

# JVM相关

## 一、JVM性能调优的6大步骤，及关键调优参数详解

### 一、JVM内存调优

![image-20210423154342856](https://gitee.com/zelen/IMG/raw/master/PicGo/image-20210423154342856.png)

**对JVM内存的系统级的调优主要目的是减少GC的频率和Full GC的次数**

1. **Full GC**

   会对整个堆进行整理，包括Yong、Tenured和Perm。Full GC需要对整个堆进行回收，所以比较慢，因此应该尽可能减少Full GC的次数

2. **导致Full GC的原因**

   1. 年老代(Tenured)被写满

      