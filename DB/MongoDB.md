---
title: MongoDB
date: 2020-07-08 18:01:31
categories: 
 - DB
 - MongoDB
tags: 
 - MongoDB
---



# MongoDB（有点乱 之后整理）

<!--more-->

## 一、数据库（Database）

- 数据库是按照数据结构来组织、存储和管理数据的仓库。我们的程序都是在内存中运行的，一旦程序运行结束或者计算机断电，程序运行中的数据都会丢失。所以我们就需要将一些程序运行的数据持久化到硬盘之中，以确保数据的安全性。而数据库就是数据持久化的最佳选择。说白了，数据库就是存储数据的仓库
- 数据库分类：
  - 关系型数据库（RDBMS）
    - MySQL、Oracle、DB2、SQLServer
    - 关系型数据库中全都是表
  - 非关系型数据库（NoSQL Not Only SQL）
    - MongoDB、Redis
    - 键值对数据库
    - 文档数据库MongoDB

## 二、MongoDB相关概念

- 业务应用场景
  - 传统的关系型数据库（如MySQL），在数据操作的“三高”需求以及应对Web2.0的网站需求面前，显得力不从心。 
  - 解释：“三高”需求： 
    -  High performance - 对数据库高并发读写的需求。
    -  Huge Storage - 对海量数据的高效率存储和访问的需求。 
    - High Scalability && High Availability- 对数据库的高可扩展性和高可用性的需求。 而MongoDB可应对“三高”需求。
  - 具体的应用场景
    1. 社交场景，使用MongoDB存储用户信息，以及用户发表的朋友圈信息，通过地理位置索引实现附近的人、地点等功能
    2. 游戏场景，使用 MongoDB 存储游戏用户信息，用户的装备、积分等直接以内嵌文档的形式存储，方便查询、高效率存储和访问
    3. 物流场景，使用 MongoDB 存储订单信息，订单状态在运送过程中会不断更新，以 MongoDB 内嵌数组的形式来存储，一次查询就能将 订单所有的变更读取出来
    4. 物联网场景，使用 MongoDB 存储所有接入的智能设备信息，以及设备汇报的日志信息，并对这些信息进行多维度的分析
    5. 视频直播，使用 MongoDB 存储用户信息、点赞互动信息等
  - 以上场景中，数据操作方面的共同特点：
    1. 数据量大
    2. 写入操作频繁（读写都很频繁）
    3. 价值较低的数据，对事务性要求不高
  - **选择MongoDB**
    - 在架构选型上，除了上述的三个特点外，如果你还犹豫是否要选择它？可以考虑以下的一些问题： 
      - 应用不需要事务及复杂 join 支持 、
      - 新应用，需求会变，数据模型无法确定，想快速迭代开发 
      - 应用需要2000-3000以上的读写QPS（更高也可以） 
      - 应用需要TB甚至 PB 级别数据存储 应用发展迅速，需要能快速水平扩展 应用要求存储的数据不丢失 应用需要99.999%高可用 应用需要大量的地理位置查询、文本查询 如果上述有1个符合，可以考虑 MongoDB，2个及以上的符合，选择 MongoDB 绝不会后悔

## 三、MongoDB简介

- MongoDB是为快速开发互联网Web应用而设计的数据库系统。MongoDB的设计目标是极简、灵活、作为Web应用栈的一部分。MongoDB的数据模型是面向文档的，所谓文档是一种类似于JSON的结构，简单理解MongoDB这个数据库中存的是各种各样的JSON。(BSON)（二进制的json）

- 三个概念： 结构类似于：MongoDB --> n * database -->n * collection -->n * document

  - 数据库（database）
    - 数据库是一个仓库，在仓库中可以存放集合
  - 集合（collection）
    - 集合类似于数组，在集合中可以存放文档
  - 文档（document）
    - 文档数据库中的最小单位，我们存储和操作的内容都是文档

- MongoDB的版本偶数版本为稳定版，奇数版本为开发版，MongoDB对于32位系统支持不佳，所以3.2版本以后没有在对32位系统的支持

- docker安装MongoDB：

  - ```shell
    docker pull mongo
    docker run -itd --name mongo -p 27017:27017 mongo --auth
    docker exec -it 2c79 mongo admin
    db.createUser({user: 'root', pwd: 'root', roles:[{ role: "root", db:"admin" }]});
    db.auth("root","root");
    ```

  - **其中 2c79 是 CONTAINER ID，即容器ID**

## 四、MongoDB技术优势总结

- JSON结构和对象模型接近，开发代码量低

- JSON的动态模型意味着更容易响应新的业务需求
- 复制集提供99.99%高可用
- 分片架构支持海量数据和无缝扩容

## 五、简单使用

### 使用insert 完成插入操作

1. 操作格式：

   - db.<集合>.insertOne(<JSON对象>)

   - db.<集合>.insertMany([<JSON1>, <JSON2>, ...<JSON3>])

2. 实例：

   - db.fruit.insertOne({name:"apple"})
   - db.fruit.insertMany([{name:"apple"},{name:"pear"},{name:"orange"}])

### 使用find查询文档

1. 关于find

   - find是MongoDB中查询数据的基本指令，相当于SQL中的SELECT
   - find返回的是游标

2. find示例

   - db.movies.find({"year":1975})  //单条件查询
   - db.movies.find({"year":1989,"title":"Batman"})  //多条件and查询
   - db.movies.find({$and:[{"title":"Batman"},{"category":"action"}]}) //and的另一种形式
   - db.movies.find({$or:[{"year":1989},{"title":"Batman"}]})  //多条件or查询
   - db.movies.find({"title":/^B/}) //按正则表达式查找

3. 对照表

   |     SQL      |               MQL               |
   | :----------: | :-----------------------------: |
   |     a=1      |              {a:1}              |
   |     a<>1     |           {a:{$ne:1}}           |
   |     a>1      |           {a:{$gt:1}}           |
   |     a>=1     |          {a:{$gte:1}}           |
   |     a<1      |           {a:{$lt:1}}           |
   |     a<=1     |          {a:{$lte:1}}           |
   | a=1 AND b=1  | {a:1,b:1}或{$and:[{a:1},{b:1}]} |
   |  a=1 OR b=1  |       {$or:[{a:1},{b:1}]}       |
   |  a IS NULL   |       {a:{$exists:false}}       |
   | a IN (1,2,3) |        {a:{$in:[1,2,3]}}        |

4.   ![](https://testingcf.jsdelivr.net/gh/znej/pic/picgo/20200608180434.png)

5.    ![](https://testingcf.jsdelivr.net/gh/znej/pic/picgo/20200609104401.png)

6.   ![](https://testingcf.jsdelivr.net/gh/znej/pic/picgo/20200609104633.png)

7. ![](https://testingcf.jsdelivr.net/gh/znej/pic/picgo/20200609105123.png)

8. ![](https://testingcf.jsdelivr.net/gh/znej/pic/picgo/20200609104914.png)

9. 使用drop删除集合

   - 使用db.<集合>.drop()来删除一个集合
   - 集合中的全部文档都会被删除
   - 集合相关的索引也会被删除
   - `db.colTOBeDropped.drop()`

10. 使用dropDatabase删除数据库

    - 使用db.dropDatabase()来删除数据库

    - 数据库相应文件也会被删除，磁盘空间将被释放

    -   ```shell
      use tempDB
      db.dropDatabase()
      show collections // No collectios
      show dbs // The db is gone
      ```

    - 
