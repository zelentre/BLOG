---
title: MySQL中的缓冲池
date:  2021-09-03 14:41:46
categories: 
 - DB
 - MySQL
tags: 
 - buffer pool
---

# MySQL中的缓冲池

## 初识缓冲池

> 缓冲池（buffer pool），简单来说就是一块内存区域。它存在的原因之一是为了避免每次都去访问磁盘，把最常访问的数据放在缓存里，提高数据的访问速度。
>
> + 是一块内存区域，当数据库操作数据的时候，把硬盘上的数据加载到buffer pool，不直接和硬盘打交道，操作的是buffer pool里面的数据
> + 数据库的增删改查都是在buffer pool上进行，和undo log/redo log/redo log buffer/binlog一起使用，后续会把数据刷到硬盘上
> + 默认大小 128M

>应用系统分层架构，为了加速数据访问，会把最常访问的数据，放在**缓存**(cache)里，避免每次都去访问数据库。
>
>操作系统，会有**缓冲池**(buffer pool)机制，避免每次访问磁盘，以加速数据的访问。
>
>MySQL作为一个存储系统，同样具有**缓冲池**(buffer pool)机制，以避免每次查询数据都进行磁盘IO。

<!--more-->

数据库中的Buffer Pool是个什么东西？其实他是一个非常关键的组件，数据库中的数据实际上最终都是要存放在磁盘文件上的，如下图所示。

![](https://gitee.com/zelen/IMG/raw/master/PicGo/20210906095718.png)

在对数据库执行增删改操作的时候，实际上主要都是针对内存里的Buffer Pool中的数据进行的，也就是实际上主要是对数据库的内存里的数据结构进行了增删改，如下图所示。

![](https://gitee.com/zelen/IMG/raw/master/PicGo/20210906134556.png)

其实每个人都担心一个事，就是你在数据库的内存里执行了一堆增删改的操作，内存数据是更新了，但是这个时候如果数据库突然崩溃了，那么内存里更新好的数据不是都没了吗？ MySQL就怕这个问题，所以引入了一个redo log机制，你在对内存里的数据进行增删改的时候，他同时会把增删改对应的日志写入redo log中，如下图。

![img](https://img2020.cnblogs.com/blog/565213/202005/565213-20200530221829286-1677247683.png)

## 小结

### 缓冲池的应用

+ 缓冲池很大程度减少了磁盘I/O带来的开销，通过将操作的数据行所在的数据页加载到缓冲池可以提高SQL的执行速度。

### 缓冲池的预读机制

+ 为了减少磁盘I/O，Innodb通过在缓冲池中提前读取多个数据页来进行优化，这种方式叫做预读。

### 缓冲池的空间管理

+ 传统的LRU方法对于缓冲池来说，会导致预读失效和缓冲池污染两种情况，因此这种传统的方式并不适用缓冲池的空间管理。

+ 基于对LRU方法的优化，MySQL设计了冷热数据分离的处理方案，将LRU链表分为热数据区和冷数据区两部分，以此解决预读失效和缓冲池污染的情况。

  

## 相关参考

[buffer pool详解 - 博客园](https://www.cnblogs.com/wasitututu/p/13612605.html)

[详解MySQL中的缓冲池buffer pool](https://www.yht7.com/news/129847)

[Buffer pool 并发控制 ](https://zhuanlan.zhihu.com/p/129567245)

[理解Mysql中的Buffer pool](https://www.cnblogs.com/wxlevel/p/12995324.html)

[缓冲池buffer pool](https://blog.csdn.net/wuhenyouyuyouyu/article/details/93377605)
