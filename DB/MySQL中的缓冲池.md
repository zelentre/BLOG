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

## 一、初识缓冲池

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



## 相关参考

[buffer pool详解 - 博客园](https://www.cnblogs.com/wasitututu/p/13612605.html)

[详解MySQL中的缓冲池buffer pool](https://www.yht7.com/news/129847)

[Buffer pool 并发控制 ](https://zhuanlan.zhihu.com/p/129567245)

[理解Mysql中的Buffer pool](https://www.cnblogs.com/wxlevel/p/12995324.html)

[缓冲池buffer pool](https://blog.csdn.net/wuhenyouyuyouyu/article/details/93377605)
