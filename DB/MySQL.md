---
title: MySQL
date: 2020-07-08 18:01:31
categories: 
 - DB
 - MySQL
tags: 
 - MySQL
---

### 数据库锁

 - 粒度小，方便用于集群环境
### 代码锁
- 粒度大，需要封装

<!-- more -->

## 微观
### 分类
#### 行锁 & 表锁
只有明确指定主键，才会执行行锁，否则执行表锁
- 无锁
```sql
     select * from user where id = -1 for update;
     #排它锁但自增主键一般从1开始  主键不存在  因此不执行锁 
```
- 行锁
```sql
 select * from user where id = 1 for update ;
 select * from user where id = 1 and name = 'kk' for update;
 #行级排它锁
```
- 表锁
```sql
 #主键不明确 锁住整张表
 select * from user where name = 'kk' for update;
 select * from user where id <> 3 for update;
```

## SQL执行顺序

![image-20220818104203296](https://gcore.jsdelivr.net/gh/znej/pic/picgo/image-20220818104203296.png)
