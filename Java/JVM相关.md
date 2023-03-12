---
title: JVM相关
date:  2021-04-23 15:29:16  
categories: 
 - Java
 - jvm
tags: 
 - jvm
---

# JVM相关

## 一、[JVM性能调优的6大步骤，及关键调优参数详解](https://zhuanlan.zhihu.com/p/222718783)

### 一、JVM内存调优

![](https://testingcf.jsdelivr.net/gh/znej/pic/picgo/image-20210423154342856.png)

**对JVM内存的系统级的调优主要目的是减少GC的频率和Full GC的次数**

1. **Full GC**

   会对整个堆进行整理，包括Yong、Tenured和Perm。Full GC需要对整个堆进行回收，所以比较慢，因此应该尽可能减少Full GC的次数

2. **导致Full GC的原因**

   - **年老代(Tenured)被写满**  调优时尽量让对象在新生代GC时被回收、让对象在新生代多存活一段时间和不要创建过大的对象及数组避免直接在旧生代创建对象 。
   - **持久代Pemanet Generation空间不足**  增大Perm Gen空间，避免太多静态对象 ， 控制好新生代和旧生代的比例
   - **System.gc()被显示调用**  垃圾回收不要手动触发，尽量依靠JVM自身的机制

   **在对JVM调优的过程中，很大一部分工作就是对于FullGC的调节**

