---
title: Redis学习
date: 2020-07-08 18:01:31
categories: 
 - DB
 - Redis
tags: 
 - Redis
---



# **Redis**

<!-- more -->

## 一、Redis初始

### redis特性
1. 速度快

   ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200303174922.png)

   - 10W OPS（每秒可以实现10万次读写 官方数字）
   - 数据存在内存中
   - 用C语言编程（50000 line）
   - 单线程模型

2. 持久化（断电不丢数据）

   - Redis所有数据保存在内存中，对数据的更新将异步地保存到磁盘上

3. 多种数据结构

   ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200303180006.png)

   - BitMaps：位图
   - HyperLogLog：超小内存唯一值计数
   - GEO：地理信息定位
   - BitMaps、HyperLogLog本质是字符串

4. 支持多种语言（Java、PHP、python、ruby、lua、node）

5. 功能丰富

   - 发布订阅
   - lua脚本
   - 事务
   - pipeline

6. 简单

   - 23000 line c code redis3 集群 （核心代码）
   - 不依赖外部库
   - 单线程模型

7. 主从复制

8. 高可用、分布式

   - Redis-Sentinel（V2.8）支持高可用
   - Redis-Cluster（V3.0）支持分布式

### Redis典型应用场景

1. 缓存系统
   - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304092708.png) 当redis中有data时直接返回data，当redis中无data-->storage,storage-->appserver、storage-->redis,以便下次访问直接走redis
2. 计数器
   - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304093255.png)转发数、评论数用缓存计数，还有视频播放数等等
3. 消息队列系统
4. 排行榜
5. 社交网络
   - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304093712.png) 粉丝数、共同关注数。。
6. 实时系统
   - 垃圾邮件系统

### redis安装

1. redis安装

   - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304094433.png)
   - **创建redis的软连接，方便以后升级**     ` ln -s redis-3.0.7 redis` 

2. 可执行文件说明

   - redis-server ---> Redis服务器
   - redis-cli ---> redis命令行客户端
   - redis-benchmark ---> 性能测试
   - redis-check-aof ---> AOF文件修复工具
   - redis-check-dump ---> RDB文件修复工具
   - redis-sentinel ---> Sentinel服务器（2.8以后）

3. 三种启动方法

   - 默认启动

     - redis-server

       ```shell
       ps -ef | grep redis
       netstat -antpl | grep redis
       redis-cli -h ip -p port ping
       redis-cli -h 127.0.0.1 -p 6379
       ```

   - 动态参数启动

     - **端口随意指定**   `redis-server --port 6380  `

   - 配置文件启动

     - redis-server configpath (configpath 配置文件的路径)

   - 比较

     - 生产环境选择配置启动
     - 单机多实例配置文件可以用端口区分

4. 简单的客户端连接

   - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304100658.png)

5. redis客户端返回值

   - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304100959.png)
   - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304101031.png)

### redis常用配置

- daemonize ---> 是否是守护进程（no|yes）
- port ---> redis对外端口号（默认6379）
- logfile ---> redis系统日志
- dir ---> redis工作目录

## 二、API的理解和使用

### 通用命令

- 通用命令

  - keys（redis里的所有的键）![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304105306.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530130939.png)

  -  **keys命令一般不再生产环境使用** **keys\*怎么用 1.热备从节点 2.scan**

  - dbsize（计算数据库大小） 

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304105824.png)

  - exists key（键是否存在） 

  ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304110033.png)

  - del key [key ...]（删除key，可以多个） 

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304110033.png)

  - expire key seconds（设置过期时间） ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304110526.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304110712.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304110829.png)

  - type key（key类型） ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304111007.png)![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304111055.png)

  - **时间复杂度大部分是O(1)**

- 数据结构和内部编码

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304112932.png)

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304140640.png)

- 单线程架构 ![redis同一时间只会执行一个命令](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304141413.png)**单线程快的原因**

  - 纯内存
  - 非阻塞IO![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304141813.png)
  - 避免线程切换和竞态消耗 ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304142116.png)

### 字符串类型

- 结构和命令

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304142451.png)**建议在100KB以内**

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530131522.png)![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304143346.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304143927.png))![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530131643.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530131719.png)![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530131750.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530131841.png)![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530131952.png)

     ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530132039.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530132132.png)

- 快速实战

  - 实现：记录网址每个用户个人主页的访问量？

    ```java
    incr userid:pageview（单线程无竞争）
    ```

  - 实现：缓存视频的基本信息（数据源在MySQL中）伪代码
    ```java
    public VideoInfo get(long id){
        String redisKey = redisPrefix + id;
        VideoInfo videoInfo = redis.get(redisKey);
        if(videoInfo == null){
            videoInf = mysql.get(id);
        }else{
            //序列化
            redis.set(redisKey,serialize(videoInfo));
        }
        return videoInfo;
    }
    ```
  - 实现：分布式id生成器 ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530132212.png)![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530132242.png)

- 内部编码

- 查漏补缺

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153215.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153316.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153335.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153356.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153417.png)

### 哈希类型

- 特点

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153436.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153511.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153534.png)

- 重要API

  - ​     ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153809.png)

    ```shell
    127.0.0.1:6379> hset user:1:info age 23
    (integer) 1
    127.0.0.1:6379>  hget user:1:info age
    "23"
    127.0.0.1:6379> hset user:1:info name ronaldo
    (integer) 1
    127.0.0.1:6379> hgetall user:1:info
    1) "age"
    2) "23"
    3) "name"
    4) "ronaldo"
    127.0.0.1:6379> hdel user:1:info age
    (integer) 1
    127.0.0.1:6379> hgetall user:1:info
    1) "name"
    2) "ronaldo"
    ```

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153641.png)

    ```shell
    127.0.0.1:6379> hgetall user:1:info
    1) "name"
    2) "ronaldo"
    3) "age"
    4) "23"
    127.0.0.1:6379> hexists user:1:info name
    (integer) 1
    127.0.0.1:6379> hlen user:1:info
    (integer) 2
    ```

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153840.png)

    ```shell
    127.0.0.1:6379> hmset user:2:info age 30 name kaka page 50
    OK
    127.0.0.1:6379> hlen user:2:info
    (integer) 3
    127.0.0.1:6379> hmget user:2:info age name
    1) "30"
    2) "kaka"
    ```

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153859.png)

    ```shell
    127.0.0.1:6379> hgetall user:2:info
    1) "age"
    2) "30"
    3) "name"
    4) "kaka"
    5) "page"
    6) "50"
    127.0.0.1:6379> hvals user:2:info
    1) "30"
    2) "kaka"
    3) "50"
    127.0.0.1:6379> hkeys user:2:info
    1) "age"
    2) "name"
    3) "page"
    ```

  - **小心使用hgetall，牢记redis是单线程（耗时问题）**

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153926.png)

  - 实现：记录网站每个用户个人主页的访问量`hincrby user:1:info pageview count`

  - 实现：缓存视频的基本信息（数据源在MySQL中）伪代码

    ```java
    public VideoInfo get(long id){
        String redisKey = redisPrefix + id;
        Map<String,String> hashMap = redis.hgetAll(redisKey);
        VideoInfo videoInfo = transferMapToVideo(hashMap);
        if(videoInfo == null){
            videoInf = mysql.get(id);
        }else{
            redis.hmset(redisKey,transferVideoToMap(hashMap));
        }
        return videoInfo;
    }
    ```

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530153943.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154007.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154023.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154055.png)

- hash vs string

- 查漏补缺

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154114.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154136.png)

### 列表类型

- 特点

  - **有序、可以重复、左右两边插入弹出**

- 重要API

  - **增**![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154205.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154220.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154238.png)

  - **删**![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154300.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154318.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154339.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154356.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154413.png)

  - **查**![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154430.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154446.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154515.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154539.png)

  - **改**![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154554.png)

    ```shell
    127.0.0.1:6379> rpush listkey a b c
    (integer) 3
    127.0.0.1:6379> lrange listkey 0 -1
    1) "a"
    2) "b"
    3) "c"
    127.0.0.1:6379> lpush listkey 0
    (integer) 4
    127.0.0.1:6379> lrange listkey 0 -1
    1) "0"
    2) "a"
    3) "b"
    4) "c"
    127.0.0.1:6379> rpop listkey
    "c"
    127.0.0.1:6379> lrange listkey 0 -1
    1) "0"
    2) "a"
    3) "b"
    ```

- 实战

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154626.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154657.png)

- 查漏补缺

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154713.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154728.png)

### 集合类型

- 特点

  - **不能插入重复元素，` srem key music`删除元素**![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154816.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154840.png)
  - **无序、无重复、集合间操作**

- 集合内API

  - **S开头**

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154923.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530154940.png)

  - **smembers：无序、小心使用**

  - **srandmember和spop：spop从集合弹出、srandmember不会破坏集合**

    ```shell
    127.0.0.1:6379> sadd user:1:follow it news his sports
    (integer) 4
    127.0.0.1:6379> smembers user:1:follow
    1) "sports"
    2) "his"
    3) "news"
    4) "it"
    127.0.0.1:6379> spop user:1:follow 
    "news"
    127.0.0.1:6379> smembers user:1:follow
    1) "sports"
    2) "his"
    3) "it"
    127.0.0.1:6379> scard user:1:follow
    (integer) 3
    127.0.0.1:6379> sismember user:1:follow entertainment
    (integer) 0
    ```

  - 实现：抽奖系统（can be spop）

  - 实现：Like、赞、踩

  - 实现：标签（同事务）

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530155001.png)

- 集合间API

  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305170220.png)
  - 实现：共同关注（sinter）
  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305170414.png)

### 有序集合类型

- 特点

  - 结构![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305170705.png)
  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305170743.png)
  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305170840.png)

- 重要API

  - **以Z开头**

  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305174533.png)

  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305174633.png)

  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305171233.png)

  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305171320.png)

  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305171437.png)

  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305172209.png)

  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305172314.png)

  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305172429.png)

  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305174255.png)

  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305174339.png)

    ```shell
    127.0.0.1:6379> zadd player:rank 1000 ronaldo 900 messi 800 c-ronaldo 600 kaka
    (integer) 4
    127.0.0.1:6379> zscore player:rank kaka
    "600"
    127.0.0.1:6379> zcard player:rank
    (integer) 4
    127.0.0.1:6379> zrank player:rank ronaldo
    (integer) 3
    127.0.0.1:6379> zrem player:rank messi
    (integer) 1
    127.0.0.1:6379> zrange player:rank 0 -1 withscores
    1) "kaka"
    2) "600"
    3) "c-ronaldo"
    4) "800"
    5) "ronaldo"
    6) "1000"
    127.0.0.1:6379> zadd player:rank 800 c-ronaldo
    (integer) 0
    127.0.0.1:6379> zadd player:rank 900 messi
    (integer) 1
    127.0.0.1:6379> zrange player:rank 0 -1
    1) "kaka"
    2) "c-ronaldo"
    3) "messi"
    4) "ronaldo"
    127.0.0.1:6379> zcount player:rank 700 901
    (integer) 2
    127.0.0.1:6379> zrangebyscore player:rank 700 901
    1) "c-ronaldo"
    2) "messi"
    127.0.0.1:6379> zremrangebyrank player:rank 0 1
    (integer) 2
    127.0.0.1:6379> zrange player:rank 0 -1
    1) "messi"
    2) "ronaldo"
    127.0.0.1:6379> zrange player:rank 0 -1 withscores
    1) "messi"
    2) "900"
    3) "ronaldo"
    4) "1000"
    ```
  
- 实战

  - 实现：排行榜![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305173811.png)

- 查漏补缺

  - ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305173929.png)

    ![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200305173949.png)

## 三、Redis客户端的使用

### Java客户端：Jedis

+ 获取jedis

  + shell（redis-cli）  Java（jedis）

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200306103814.png)

  + ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200306103935.png)

  + Jedis直连

    ```java
    // 1.生成一个Jedis对象，这个对象负责和指定redis节点进行通信
        Jedis jedis = new Jedis("127.0.0.1",6379);
    // 2.jedis执行set操作
        jedis.set("hello","world");
    // 3.jedis执行get操作，value="world"
        String value = jedis.get("hello");
    ```

    **Jedis(String host, int port, int connectionTimeout, int soTimeout)**

    + host：Redis节点的所在机器的IP
    + port：Redis节点的端口
    + connectionTimeout：客户端连接超时
    + soTimeout：客户端读写超时

+ jedis的基本使用

  ```java
  // 1.String
  // 输出结果：ok
  jedis.set("hello","world");
  // 输出结果：world
  jedis.get("hello");
  // 输出结果：1
  jedis.incr("counter");
  
  // 2.hash
  jedis.hset("myhash","f1","v1");
  jedis.hset("myhash","f2","v2")；
  // 输出结果：{f1=v1, f2=v2}
  jedis.hgetall("myhash");
  
  // 3.list
  jedis.rpush("mylist","1");
  jedis.rpush("mylist","2");
  jedis.rpush("mylist","3");
  // 输出结果：[1,2,3]
  jedis.lrange("mylist",0,-1);
  
  // 4.set
  jedis.sadd("myset","a");
  jedis.sadd("myset","b");
  jedis.sadd("myset","a");
  // 输出结果：[b, a]
  jedis.smembers("myset");
  
  // 5.zset
  jedis.zadd("myzset",99,"tom");
  jedis.zadd("myzset",66,"peter");
  jedis.zadd("myzset",33,"james");
  // 输出结果：[[["james"],33.0],[["peter"],66.0],[["tom"],99.0]]
  jedis.zrangeWithScores("myzset",0,-1);
  ```

+ jedis连接池使用

  + Jedis直连

    + ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200306113555.png)

  + Jedis连接池

    + ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200306113844.png)

  + 方案对比

    + ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200306134849.png)

  + JedisPool使用

    + 初始化Jedis连接池，通常来讲JedisPool是单例的。

      ```java
      GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
      JedisPool jedisPool = new JedisPool(poolConfig,"127.0.0.1","6379");
      Jedis jedis = null;
      try{
          // 1.从连接池获取jedis对象
          jedis = jedisPool.getResource();
          // 2.执行操作
          jedis.set("hello","world");
      }catch(Exception e){
          e.printStackTrace();
          // Logger logger = LoggerFactory.getLogger(getClass().getName());
          //logger.error(e.getMessage(),e);
      }finally{
          if(jedis != null)
              // 如果使用JedisPool，close操作不是关闭连接，代表归还连接池
              jedis.close();
      }
      ```

  + Jedis配置优化

    + ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200306141709.png)
    + ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200306142536.png)
    + 适合的maxTotal（比较难确定）
      + 命令平均执行时间 0.1ms = 0.001s。
      + 业务需要50000 QPS。
      + maxTotal理论值 = 0.001*50000 = 50个。实际值要偏大一些。
    + maxTotal（需要考虑）
      + 业务希望redis并发量
      + 客户端执行命令时间
      + redis资源：例如 nodes（例如应用个数）* maxTotal 是不能超过redis的最大连接数。（config get maxclients）
    + 适合的maxIdle和minIdle
      + 建议maxIdle = maxTotal
        + 减少创建新连接的开销
      + 建议预热minIdle
        + 减少第一次启动后的新连接开销
    + 常见问题
      - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200306144528.png)
    + 解决思路
      - 慢查询阻塞：池子连接都被hang住。
      - 资源池参数不合理：例如QPS高、池子小。
      - 连接泄密（没有close()）：此类问题比较难定位，例如client list、netstat等，最重要的是代码。
      - DNS异常等。

## 四、Redis其他功能

### 慢查询

1. 生命周期

   - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309095118.png)

2. 两个配置

   ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309095543.png)

   + slowlog-max-len
     1. 先进先出队列
     2. 固定长度
     3. 保存在内存内
   + slowlog-log-slower-than
     1. 慢查询阈值（单位：微秒）
     2. slowlog-log-slower-than=0，记录所有命令
     3. slowlog-log-slower-than<0，不记录任何命令
   + 配置方法
     1. 默认值
        - `config get slowlog-max-len = 128`
        - `config get slowlog-log-slower-than = 10000`
     2. 修改配置文件重启
     3. 动态配置
        - `config set slowlog-max-len 1000`
        - `config set slowlog-log-slower-than 1000`

3. 三个命令

   1. slowlog get [n]：获取慢查询队列
   2. slowlog len：获取慢查询队列长度
   3. slowlog reset：清空慢查询队列

4. 运维经验

   1. slowlog-max-len不要设置过大，默认10ms，通常设置1ms
   2. slowlog-log-slower-than不要设置国国小，通常设置1000左右
   3. 理解命令生命周期
   4. 定期持久化慢查询

### pipeline

- 什么是流水线

  - **一次网路命令通信模型**![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304171244.png)

  - **批量网络命令通信模型**![img](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200304171152.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309103325.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309103738.png)

- 客户端使用

  - 没有pipeline

    ```java
    Jedis jedis = new Jedis("127.0.0.1",6379);
    for(int i = 0; i < 10000; i++){
        jedis.hset("hashKey:"+i, "field"+i, "value"+i);
        // 1W hset ---> 50s（预估）
    }
    ```

  - 使用pipeline

    ```java
    Jedis jedis = new Jedis("127.0.0.1",6379);
    for(int i = 0; i < 100; i++){
        Pipeline pipeline = jedis.pipelined();
        for(int j = i*100; j < (i+1)*100; j++){
            pipeline.hset("hashKey:"+j, "field"+j, "value"+j);
        }
        pipeline.syncAndReturnAll();
        //1W hset ---> 0.7s（预估）
    }
    ```

    

- 与原生操作对比

  - **与原生相比 pipeline是非原子**

- 使用建议

  1. 注意每次pipeline携带数据量
  2. pipeline每次只能作用在一个Redis节点上
  3. M操作与pipeline区别

### 发布订阅

- 角色

  - 发布者（publisher）
  - 订阅者（subscriber）
  - 频道（channel）

- 模型

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309111739.png)
  - **订阅这可以订阅多个频道**

- API

  - publish

    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309112125.png)

  - subscribe

    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309112203.png)

  - unsubscribe

    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309112241.png)

  - 其他

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309112346.png)

- 发布订阅与消息队列

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309112429.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309112608.png)

### Bitmap

- 位图

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309113011.png)

- 相关命令

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309143205.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309143831.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309143926.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309144012.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309144120.png)

- 独立用户统计

  1. 使用set和Bitmap
  2. 1亿用户，5千万独立

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309144549.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309144651.png)
  - **使用经验**
    - 1. type=string，最大512MB
      2. 注意setbit时的偏移量，可能有较大耗时
      3. 位图不是绝对好。

### HyperLogLog

- 新的数据结构？

  1. 基于HyperLogLog算法：极小空间完成独立数量统计

  2. 本质还是字符串

     ```shell
     127.0.0.1:6379> type hyperloglog_key
     string
     ```

- 三个命令

  1. `pfadd key element [element ...]`：向hyperloglog添加元素
  2. `pfcount key [key ...]`：计算hyperloglog的独立总数
  3. `pfmerge destkey sourcekey [sourcekey ...]`：合并多个hyperloglog

- 内存消耗

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309151432.png)

- 使用经验

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309151656.png)

### GEO

- GEO是什么

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309151922.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309152004.png)

- 5个城市经纬度

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309152029.png)

- 相关命令

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309152309.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309152422.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309152513.png)

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309152538.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309152646.png)

- 相关说明

  1. since 3.2+
  2. type geoKey = zset
  3. 没有删除API：zrem key member

## 五、Redis持久化的取舍和选择

### 持久化的作用

- 什么是持久化
  - redis所有数据保持在内存中，对数据的更新将异步地保存到磁盘上
- 持久化的实现方式
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309154132.png)

### RDB

- 什么是RDB

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309154714.png)

- 触发机制-主要三种方式

  - save（同步）

    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309155136.png)
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309155204.png)
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309155246.png)

  - bgsave（异步）

    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20220530160629.png)
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309164246.png)
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309164333.png)
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309164353.png)

  - 自动

    - **自动生成RDB**

      ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309165036.png)

      ```shell
      # 配置文件
      save 900 1
      save 300 10
      save 60 10000
      dbfilename dump.rdb
      dir ./
      stop-writes-on-bgsave-error yes
      rdbcompression yes
      rdbchecksum yes
      ```
      ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309165940.png)

- 触发机制-不容忽视方式

  1. 全量复制
  2. debug reload
  3. shutdown

- 总结

  1. RDB是Redis内存到硬盘的快照，用于持久化
  2. save通常会阻塞Redis
  3. bgsave不会阻塞Redis，但是会fork新进程
  4. save自动配置满足任一就会被执行
  5. 有些触发机制不容忽视

### AOF

- RDB现存问题
  - 耗时、耗性能
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309173446.png)
  - 不可控、丢失数据
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309173612.png)
- 什么是AOF
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309173720.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309173807.png)
- AOF三种策略
  - always
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309174020.png)
  - everysec
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309174135.png)
  - no
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309174219.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309174304.png)
- AOF重写
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309174657.png)
  - 作用：
    - 减少硬盘占用量
    - 加速恢复速度
  - AOF重写实现两种方式
    - bgrewriteaof
      - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309175535.png)
    - AOF重写配置
      - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309175607.png)
      - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309180145.png)
      - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309180238.png)
      - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309180304.png)
      - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200309180622.png)

### RDB和AOF的抉择

- RDB和AOF比较

  |    命令    |  RDB   |     AOF      |
  | :--------: | :----: | :----------: |
  | 启动优先级 |   低   |      高      |
  |    体积    |   小   |      大      |
  |  恢复速度  |   快   |      慢      |
  | 数据安全性 | 丢数据 | 根据策略决定 |
  |    轻重    |   重   |      轻      |

- RDB最佳策略

  - “关”
  - 集中管理
  - 主从，从开？

- AOF最佳策略

  - "开"：缓存和存储
  - AOF重写集中管理
  - everysec

- 最佳策略

  - 小分片
  - 缓存或者存储
  - 监控（硬盘、内存、负载、网络）
  - 足够的内存

## 六、常见的持久化开发运维问题

### fork操作

1. 同步操作
2. 与内存量息息相关：内存越大,耗时越长（与机器类型有关）
3. info：latest_fork_usec

- 改善fork
  1. 优先使用物理机或者高效支持fork操作的虚拟化技术
  2. 控制Redis实例最大可用内存：maxmemory
  3. 合理配置Linux内存分配策略：vm.overcommit_memory = 1
  4. 降低fork频率：例如放宽AOF重写自动触发时机，不必要的全量复制

### 子进程开销和优化

1. CPU：
   - 开销：RDB和AOF文件生成，属于CPU密集型
   - 优化：不做CPU绑定，不和CPU密集型部署
2. 内存
   - 开销：fork内存开销，copy-on-write
   - 优化：echo never > /sys/kernel/mm/transparent_hugepage/enabled
3. 硬盘
   - 开销：AOF和RDB文件写入，可以结合iostat，iotop分析
   - 优化：
     1. 不要和高硬盘负载服务部署一起：存储服务、消息队列等
     2. no-appendfsync-on-rewrite = yes
     3. 根据写入量决定磁盘类型：例如ssd
     4. 单机多实例持久化文件目录可以考虑分盘

### AOF追加阻塞

- ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310114122.png)
- AOF阻塞定位
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310134905.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310134922.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310134939.png)

## 七、Redis复制的原理与优化

### 什么是主从复制

- 单机问题：

  - 机器故障
  - 容量瓶颈
  - QPS瓶颈

- 主从复制的作用

  - **一主一从**

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310140307.png)

  - **一主多从**

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310140451.png)

  - **作用**

    - 数据副本
    - 扩展读性能

  - **总结**

    1. 一个master可以有多个slave
    2. 一个slave只能有一个master
    3. 数据流向是单向的，master到slave

### 复制的配置

- slaveof命令

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310141243.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310141458.png)

- 配置

  ```shell
  slaveof ip port
  slave-read-only yes
  ```

- **比较**

  | 方式 |    命令    |   配置   |
  | :--: | :--------: | :------: |
  | 优点 |  无需重启  | 统一配置 |
  | 缺点 | 不便于管理 | 需要重启 |

### 全量复制和部分复制

- **全量复制**

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310152018.png)

- **全量复制开销**

  1. bgsave时间
  2. RDB文件网络传输时间
  3. 从节点清空数据时间
  4. 从节点加载RDB的时间
  5. 可能的AOF重写时间

- **部分复制**

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310160012.png)

### 故障处理

- **主从结构-故障转移**

  - slave宕掉

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310160740.png)

  - master宕掉

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310160902.png)

### 开发运维常见问题

1. 读写分离

   ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310161840.png)

   1. 读流量分摊到从节点
   2. 可能遇到的问题：
      - 复制数据延迟（发生阻塞）
      - 读到过期数据
        - [删除过期数据的策略](https://blog.csdn.net/zmflying8177/article/details/104215634)
          1. 懒惰性策略（操作key时才会检查key是否过期，过期才删除，然后返回null）
          2. 定时删除策略（定时扫描所有键值对，发现过期数据立即删除）
      - 从节点故障

2. 主从配置不一致

   1. 例如maxmemory不一致：丢失数据
   2. 例如数据结构优化参数（例如hash-max-ziplist-entries）：内存不一致

3. 规避全量复制

   1. 第一次全量复制
      - 第一次不可避免
      - 小主节点、低峰
   2. 节点运行ID不匹配
      - 主节点重启（运行ID变化）
      - 故障转移，例如哨兵或集群
   3. 复制积压缓冲区不足
      - 网络中断，部分复制无法满足
      - 增大复制缓冲区配置rel_backlog_size，网络“增强”

4. 规避复制风暴

   1. 单主节点复制风暴：

      ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310173149.png)

      - 问题：主节点重启，多从节点复制
      - 解决：更换复制拓扑

   2. 单机器复制风暴

      ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310173236.png)

      - 机器宕机后，大量全量复制
      - 主节点分散多机器

## 八、Redis Sentinel

### 主从复制高可用？

- 手动故障转移

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200310160902.png)

- 写能力和存储能力受限

### 架构说明

- ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200311092857.png)

- ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200311093130.png)

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200311093208.png)

### 安装配置

1. 配置开启主从节点
   - 主节点
   - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200311162522.png)
   - 从节点
   - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200311162637.png)
   - 主要配置
   - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200311162829.png)
2. 配置开启sentinel监控主节点。（sentinel是特殊的redis）
3. 实际应该多机器
4. 详细配置节点

### 客户端连接

- 请求流程

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200311165122.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200311165212.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200311165229.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200311165336.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200311165359.png)
  - **客户端接入流程：**
    1. Sentinel地址集合
    2. masterName
    3. 不是代理模式

- jedis

  ```java
  JedisSentinelPool sentinelPool = new JedisSentinelPool(masterName,sentinelSet,poolConfig,timeout);
  Jedis jedis = null;
  try{
      // jedis command
      jedis = sentinelPool.getResource();
  } catch(Exception e){
      logger.error(e.getMessage(),e);
  } finally{
      if(jedis != null){
          jedis.close();
      }
  }
  ```

### 实现原理

- 故障转移演练

  1. 客户端高可用观察

  2. 服务端日志分析：数据节点和sentinel节点

     ```java
     public static void main(String[] args) {
     	private static Logger logger =
             LoggerFactory.getLogger(RedisSentinelTest.class);
     
         String masterName = "myMaster";
         Set<String> sentinels = new HashSet<>();
         sentinels.add("127.0.0.1:26379");
         sentinels.add("127.0.0.1:26380");
         sentinels.add("127.0.0.1:26381");
     
         JedisSentinelPool jedisSentinelPool = new
             JedisSentinelPool(masterName,sentinels);
         
         int counter = 0;
         while (true){
             counter++;
             Jedis jedis = null;
             try {
                 jedis = jedisSentinelPool.getResource();
                 int index = new Random().nextInt(100000);
                 String key = "k-" + index;
                 String value = "v-" + index;
                 jedis.set(key,value);
                 if(counter % 100 == 0){
                     logger.info("{} value is {}", key, jedis.get(key));
                 }
                 TimeUnit.MILLISECONDS.sleep(10);
              } catch (Exception e){
                 logger.error(e.getMessage(),e);
              } finally {
                 if(jedis != null){
                     jedis.close();}
              }
      }
     }
     ```
     
  
- 三个定时任务

  1. 每10秒每个sentinel对master和slave执行info

     - 发现slave节点
     - 确认主从关系

     ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200312113712.png)

  2. 每2秒每个sentinel通过master节点的channel交换信息（pub/sub）

     - 通过 `_sentinel_:hello`频道交换
     - 交换对节点的“看法”和自身信息

     ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200312134718.png)

  3. 每1秒每个sentinel对其他sentinel和redis执行ping

     ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200312153224.png)

- 主观下线和客观下线

  - 配置示例

    ```shell
    sentinel monitor <masterName> <ip> <port> <quorum>
    sentinel monitor myMaster 127.0.0.1 6379 2
    sentinel down-after-milliseconds <masterName> <timeout>
    sentinel down-after-milliseconds myMaster 30000
    ```

  - 主观下线：每个sentinel节点对Redis节点失败的“偏见”

  - 客观下线：所有sentinel节点对Redis节点失败“达成共识”（超过quorum个统一）

    `sentinel is-master-down-by-addr`

- 领导者选举

  - 原因：只有一个sentinel节点完成故障转移
  - 选举：通过`sentinel is-master-down-by-addr`命令都希望成为领导者
    1. 每个做主观下线的sentinel节点向其他sentinel节点发送命令，要求将它设置为领导者
    2. 收到命令的sentinel节点如果没有同意通过其他sentinel节点发送的命令，那么将同意该请求，否则拒绝
    3. 如果该sentinel节点发现自己的票数已经超过sentinel集合半数且超过quorum，那么它将成为领导者
    4. 如果此过程有多个sentinel节点成为了领导者，那么将等待一段时间重新进行选举

- 故障转移（sentinel领导者节点完成）

  1. 从slave节点中选出一个“合适”节点作为新的master节点
  2. 对上面的slave节点执行slaveof no one 命令让其成为master节点
  3. 向剩余的slave节点发送命令，让它们成为新master节点的slave节点，复制规则和parallel-syncs参数有关
  4. 更新对原来master节点配置为slave，并保持着对其“关注”，当其恢复后命令它去复制新的master节点

- 选择“合适的”slave节点

  1. 选择slave-priority（slave节点优先级）最高的slave节点，如果存在则返回，不存在则继续
  2. 选择复制偏移量最大的slave节点（复制的最完整），如果存在则返回，不存在则继续
  3. 选择runId最小的slave节点

### 开发运维常见问题

- 节点运维

  - 节点下线

    - 主节点

      `sentinel failover <masterName>`

      ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200312165933.png)

    - 从节点：临时下线还是永久下线，例如是否做一些清理工作，但是要考虑读写分离的情况

    - sentinel节点：类同于从节点

    - 机器下线：例如过保等情况

    - 机器性能不足：例如CPU、内存、硬盘、网络等

    - 机器自身故障：例如服务不稳定等

  - 节点上线
    - 主节点：sentinel failover进行替换
    - 从节点：slaveof即可，sentinel节点可以感知
    - sentinel节点：参考其他sentinel节点启动即可

- 高可用读写分离

  - 从节点的作用
    1. 副本：高可用的基础
    2. 扩展：读能力
  - 三个“消息”
    - +switch-master：切换主节点（从节点晋升主节点）
    - +convert-to-slave：切换从节点（原主节点降为从节点）
    - +sdown：主观下线

### 总结

- redis sentinel是redis的高可用实现方案：故障发现、故障自动转移、配置中心、客户端通知
- redis sentinel从redis2.8版本开始才正式生产可用，之前版本生产不可用
- 尽可能在不同物理机上部署redis sentinel所有节点
- redis sentinel中的sentinel节点个数应该大于等于3且最好为奇数
- redis sentinel中的数据节点与普通数据节点没有区别
- 客户端初始化时连接的是sentinel节点集合，不再是具体的redis节点，但sentinel只是配置中心不是代理
- redis sentinel 通过三个定时任务实现了sentinel节点对于主节点、从节点、其余sentinel节点的监控
- redis sentinel在对节点做失败判定时分为主观下线和客观下线
- 看懂redis sentinel故障转移日志对于redis sentinel以及问题排查非常有帮助
- redis sentinel实现读写分离高可用可以依赖sentinel节点的消息通知，获取redis数据节点的状态变化

## 九、Redis Cluster

### 呼唤集群

- **扩容最好翻倍扩容**

- 使用集群原因：
  1. 并发量：最大10W/s，业务需求更多呢
  2. 数据量：机器内存有限，业务需求大呢
- 解决方法：
  - 配置强悍的机器（×）
  - 分布式：简单的认为 加机器
- 集群：规模化需求
  - 并发量：QPS
  - 数据量

### 数据分布

- ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313101436.png)

- 顺序分布和哈希分布

  - 顺序分布

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313110955.png)

  - 哈希分布（例如节点取模）

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313111241.png)

- 数据分布对比

  | 分布方式 |                            特点                            |                   典型产品                    |
  | :------: | :--------------------------------------------------------: | :-------------------------------------------: |
  | 哈希分布 | 数据分散度高  键值分布业务无关  无法顺序访问  支持批量操作 | 一致性哈希Memcache Redis Cluster 其他缓存产品 |
  | 顺序分布 |  数据分散度易倾斜  键值业务相关  可顺序访问  支持批量操作  |               BigTable   HBase                |

- 哈希分布

  - 节点取余分区：hash(Key)%nodes

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313111241.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313134742.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313135502.png)

  - 节点取余：

    - 客户端分片：哈希+取余
    - 节点伸缩：数据节点关系变化，导致数据迁移
    - 迁移数量和添加节点数量有关：建议翻倍扩容

  - 一致性哈希分区

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313140707.png)

    - n1 和 n2之间的key的数据将偏移到n5

      ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313141030.png)

    - 一致性哈希
      - 客户端分片：哈希+顺时针（优化取余）
      - 节点伸缩：只影响邻近节点，但是还是有数据迁移
      - 翻倍伸缩：保证最小迁移数据和负载均衡

  - 虚拟槽分区

    - 预设虚拟槽：每个槽映射一个数据子集，一般比节点数大

    - 良好的哈希函数：例如CRC16

    - 服务端管理节点、槽、数据：例如Redis Cluster

    - 虚拟槽分配

      ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313142046.png)

### 搭建集群

- 基本架构

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313142530.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313142550.png)

- Redis Cluster架构

  - 节点

    - cluster-enabled:yes  即集群启动

      ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313142550.png)

  - meet

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313151106.png)

  - 指派槽

    - 客户端与指派槽

      ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313151255.png)

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313151145.png)

  - 复制

  - 特性

    - 复制
    - 高可用
    - 分片

- Redis Cluster安装

  - 原生命令安装

    1. 配置开启节点

       ```shell
       port ${port}
       daemonize yes
       dir "/opt/redis/redis/data/"
       dbfilename "dump-${port}.db"
       logfile "${port}.log"
       cluster-enabled yes
       cluster-config-file nodes-${port}.conf
       
       redis-server redis-7000.conf
       redis-server redis-7001.conf
       redis-server redis-7002.conf
       redis-server redis-7003.conf
       redis-server redis-7004.conf
       redis-server redis-7005.conf
       ```

       - cluster节点主要配置

         ```shell
         cluster-enabled yes
         cluster-node-timeout 15000
         cluster-config-file "nodes.conf"
         cluster-require-full-coverage yes
         ```

    2. meet

       - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313161429.png)

    3. 指派槽

       - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313161916.png)

    4. 主从

       - 设置主从

         - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200313162242.png)

    5. 总结

       - 理解redis cluster架构
   - 生产环境不使用
    
  - 官方工具安装

    - 可以查看这个博客[Redis-cluster集群](https://www.cnblogs.com/cqming/p/11191079.html)

    - 高效、准确
  - 生产环境可以使用
  
  - 其他：可视化部署

### 集群伸缩

- 伸缩原理

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316135757.png)

- 扩容集群

  - 准备新节点

    - 集群模式

    - 配置和其他节点统一

    - 启动后是孤儿节点

      `redis-server conf/redis-6385.conf`

      `redis-server conf/redis-6386.conf`

  - 加入集群

    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316140616.png)
    - 作用：
      - 为它迁移槽和数据实习扩容
      - 作为从节点负责故障转移
    - 加入集群-redis-trib.rb
      - `redis-trib.rb add-node new_host:new_port existing_host:existing_port --slave --master-id <arg>`
      - `redis-trib.rb add-node 127.0.0.1:6385 127.0.0.1:6379`
      - 建议使用redis-trib.rb能够避免新节点已经加入了其他集群，造成故障

  - 迁移槽和数据

    - 槽迁移计划
      - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316143153.png)
    - 迁移数据
      1. 对目标节点发送：`cluster setslot {slot} importing {sourceNodeId}`命令，让目标节点准备导入槽的数据
      2. 对源节点发送：`cluster setslot {slot} migrating {targetNodeId}`命令，让源节点准备迁出槽的数据
      3. 源节点循环执行`cluster getkeysinslot {slot} {count}`命令，每次获取count个属于槽的键
      4. 在源节点上执行`migrate {targetId} {targetPort} key 0 {timeout}`命令把指定key迁移
      5. 重复执行步骤3~4知道槽下所有的键数据迁移到目标节点
      6. 向集群内所有主节点发送`cluster setslot {slot} node {tartgetNodeId}`命令，通知槽分配给目标节点 
      7. ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316144823.png)
    - 添加从节点

- 缩容集群

### 客户端路由

- moved重定向

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316152159.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316152247.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316152428.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316152657.png)

- ask重定向

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316152940.png)
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316153110.png)
  - moved和ask
    - 两者都是客户单重定向
    - moved：槽已经确定迁移
    - ask：槽还在迁移中

- smart客户端

  - smart客户端原理（追求性能）

    1. 从集群中选一个可运行节点，使用cluster slots初始化槽和节点映射
    2. 将cluster slots的结果映射到本地，为每个节点创建JedisPool
    3. 准备执行命令
       - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316154352.png)

  - smart客户端使用：JedisCluster

    - JedisCluster基本使用

      ```java
      Set<HostAndPort> nodeList = new HashSet<HostAndPort>();
      nodeList.add(new HostAndPort(HOST1,PORT1));
      nodeList.add(new HostAndPort(HOST1,PORT2));
      nodeList.add(new HostAndPort(HOST1,PORT3));
      nodeList.add(new HostAndPort(HOST1,PORT4));
      nodeList.add(new HostAndPort(HOST1,PORT5));
      nodeList.add(new HostAndPort(HOST1,PORT6));
      JedisCluster redisCluster = new JedisCluster(nodeList,timeout,poolConfig);
      redisCluster.command ...
      ```

      - 使用技巧：
        1. 单例：内置了所有节点的连接池
        2. 无需手动接环连接池
        3. 合理设置commons-pool

    - 整合spring

      - [jedisCluster](https://github.com/zelentre/tem/blob/master/src/main/java/com/zelentre/jedis/JedisClusterFactory.java)

    - 多节点命令实现

      ```java
      Map<String, JedisPool> jedisPoolMap = jedisCluster.getClusterNodes();
      for(Entry<String,JedisPool> entry : jedisPoolMap.entrySet()){
          //获取每个节点的Jedis连接
          Jedis jedis = entry.getValue.getResource();
          //只删除主节点数据
          if(!isMaster(jedis)){
              continue;
          }
          //finally close
      }
      ```

    - 批量命令实现

      - mget mset必须在一个槽上

      - 四种批量优化的方法

        1. 串行mget

           - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316171442.png)

        2. 串行IO

           - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316171622.png)

        3. 并行IO

           - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316171711.png)

        4. hash_tag

           - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200316172031.png)

        5. 总结

           |   方案   |                优点                |                  缺点                  |      网络IO       |
           | :------: | :--------------------------------: | :------------------------------------: | :---------------: |
           | 串行mget |     编程简单  少量keys满足需求     |          大量keys请求延迟严重          |      O(keys)      |
           |  串行IO  |     编程简单  少量节点满足需求     |            大量node延迟严重            |     O(nodes)      |
           |  并行IO  | 利用并行特性  延迟取决于最慢的节点 |        编程复杂  超时定位问题难        | O(max_slow(node)) |
           | hash_tag |              性能最高              | 读写增加tag成本  tag分布易出现数据倾斜 |       O(1)        |


### 故障转移

- 故障发现
  - 通过ping/pong消息实现故障发现：不需要sentinel
  - 主观下线
    - 定义：某个节点认为另一个节点不可用，“偏见”
    - 流程：![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200317091147.png)
  - 客观下线
    - 定义：当半数以上持有槽的主节点都标记某几点主观下线
    - 逻辑流程：![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200317091350.png)
    - 流程：![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200317091506.png)
      - 通知集群内所有节点标记故障节点为客观下线
      - 通知故障节点的从节点触发故障转移流程
- 故障恢复
  - 资格检查
    - 每个从节点检查与故障主节点的断线时间
    - 超过`cluster-node-timeout`*`cluster-slave-validity-factor`取消资格（都是默认值得话为150s）
    - cluster-slave-validity-factor：默认是10
  - 准备选举时间
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200317094316.png)
  - 选举投票
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200317094419.png)
  - 替换主节点
    1.  当前从节点取消复制变为主节点（slave no one）
    2. 执行clusterDelSlot撤销故障主节点负责的槽，并执行clusterAddSlot把这些槽分配给自己
    3. 向集群广播自己的pong消息，表明已经替换了故障从节点
- 故障转移演练
  - 步骤：
    1. 执行kill -9节点模拟宕机
    2. 观察客户端故障恢复时间

### 常见问题

- 集群完整性
  - cluster-require-full-coverage默认为yes
    - 集群中16384个槽全部可用：保证集群完整性
    - 节点故障或者正在故障转移：（error）CLUSTERDOWN The cluster is down
  - 大多数业务无法容忍，cluster-require-full-coverage建议设置为no
- 带宽消耗
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200317104154.png)
  - 消息发送频率：节点发现与其他节点最后通信时间超过cluster-node-timeout/2时会直接发送ping消息
  - 消息数据量：slots槽数组（2KB空间）和整个集群1/10的状态数据（10个节点状态数据约1KB）
  - 节点部署的机器规模：集群分布的机器越多且每台机器划分的节点数越均匀，则集群内整体的可用带宽越高
  - 例子：
    - 规模：节点200个、20台物理机（每台10个节点）
    - cluster-node-timeout = 15000，ping/pong带宽为25ＭB
    - cluster-node-timeout = 20000，ping/pong带宽为25ＭB
  - 优化：
    - 避免“大”集群：避免多业务使用一个集群，大业务可以多集群
    - cluster-node-timeout：带宽和故障转移速度的均衡
    - 尽量均匀分配到多机器上：保证高可用和带宽
- Pub/Sub广播
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200317112845.png)
  - 问题：publish在集群每个节点广播：加重宽带
  - 解决：单独“走”一套Redis Sentinel
- 集群倾斜
  - 数据倾斜：内存不均
    - 节点和槽分配不均
      - redis-trib.rb info ip:port查看节点、槽键值分布
      - redis-trib.rb rebalance ip:port进行均衡（谨慎使用）
    - 不同槽对应键值数量差异较大
      - CRC16正常情况下比较均匀
      - 可能存在hash_tag
      - cluster countkeysinslot {slot}获取槽对应键值个数
    - 包含bigkey
      - bigkey：例如大字符串、几百万的元素的hash、set等
      - 从节点：redis-cli --bigkeys
      - 优化：优化数据结构
    - 内存相关配置不一致
      - hash-max-ziplist-value、set-max-intset-entries等
      - 优化：定期“检查”配置一致性
  - 请求倾斜：热点
    - 热点key：重要的key或者bigkey
    - 优化：
      - 避免bigkey
      - 热键不要用hash_tag
      - 当一致性不高时，可以用本地缓存+MQ
- 集群读写分离
  - 只读连接：集群模式的从节点不接受任何读写请求
    - 重定向到负责槽的主节点
    - readonly命令可以读：连接级别命令
  - 读写分离：更加复杂
    - 同样的问题：复制延迟、读取过期数据、从节点故障
    - 修改客户端：cluster slave {nodeId}
- 数据迁移（在线/离线）
  - 官方迁移工具：redis-trib.rb import
    - 只能从单机迁移到集群
    - 不支持在线迁移：source需要停写
    - 不支持断点续传
    - 单线程迁移：影响速度
  - 在线迁移：
    - 唯品会 redis-migrate-tool
    - 豌豆荚 redis-port
- 集群vs单机
  - 集群限制
    - key批量操作支持有限：例如mget、mset必须在一个slot
    - key事务和Lua支持有限：操作的key必须在一个节点
    - key是数据分区的最小粒度：不支持bigkey分区
    - 不支持多数据库：集群模式下只有一个db 0
    - 复制只支持一层：不支持树形复制结构
  - 分布式redis不一定好
    1. redis cluster：满足容量和性能的扩展，很多业务“不需要”
       - 大多数时客户端性能会“降低”
       - 命令无法跨节点使用：mget、keys、scan、flush、sinter等
       - Lua和事务无法跨节点使用
       - 客户端维护更复杂：SDK和应用本身消耗（例如更多的连接池）
    2. 很多场景redis cluster已经足够好了

### 总结

- redis cluster数据分区规则采用虚拟槽方式（16384个槽），每个节点负责一部分槽和相关数据，实现数据和请求的负载均衡
- 搭建集群划分四个步骤：准备节点、节点握手、分配槽、复制。redis-trib.rb工具用于快速搭建集群
- 集群伸缩通过在节点之间移动槽和相关数据实现
  - 扩容时根据槽迁移计划把槽从源节点迁移到新节点
  - 收缩时如果下线的节点有负责的槽需要迁移到其他节点，在通过cluster forget命令让集群内所有节点忘记被下线节点
- 使用smart客户端操作集群达到通信效率最大化，客户端内部负责计算维护键 -> 槽 -> 节点的映射，用于快速定位到目标节点
- 集群自动故障转移过程分为故障发现和节点恢复。节点下线分为主观下线和客观下线，当超过半数主节点认为故障节点为主观下线时标记他为客观下线状态。从节点负责对客观下线的主节点触发故障恢复流程，保证集群的可用性
- 开发运维常见问题包括：超大规模集群带宽消耗，pub/sub广播问题，集群倾斜问题，单机和集群对比等

## 十、缓存设计与优化

### 缓存的受益与成本

- 受益
  1. 加速读写
     - 通过缓存加速读写速度：CPU L1/L2/L3 Cache、Linux page Cache加速硬盘读写、浏览器缓存、Ehcache缓存数据库结果
  2. 降低后端负载
     - 后端服务器通过前端缓存降低负载：业务端使用Redis降低后端MySQL负载等
- 成本
  1. 数据不一致：缓存层和数据层有时间窗口不一致，和更新策略有关
  2. 代码维护成本：多了一层缓存逻辑
  3. 运维成本：例如Redis Cluster
- 使用场景
  1. 降低后端负载
     - 对高消耗的SQL：join结果集/分组统计结果缓存
  2. 加速请求响应
     - 利用Redis/Memcache优化IO响应时间
  3. 大量写合并为批量写
     - 如计数器先Redis累加在批量写DB

### 缓存跟新策略

1. LRU/LFU/FIFO算法剔除：例如：maxmemory-policy

2. 超时剔除：例如expire

3. 主动更新：开发控制生命周期

4. 比较：

   |       策略       | 一致性 | 维护成本 |
   | :--------------: | :----: | :------: |
   | LRU/LIRS算法剔除 |  最差  |    低    |
   |     超时剔除     |  较差  |    低    |
   |     主动更新     |   强   |    高    |

5. 建议：

   1. 低一致性：最大内存和淘汰策略
   2. 高一致性：超时剔除和主动更新结合，最大内存和淘汰策略兜底

### 缓存粒度控制

![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200318155735.png)

1. 从MySQL获取用户信息：`select * from user where id={id}`
2. 设置用户信息缓存：`set user:{id}` ` select * from user where id={id}`
3. 缓存粒度：
   - 全部属性：`set user:{id}` `select*from user where id={id}`
   - 部分重要属性：`set user:{id}` `select importantCoumn1,..importantComnK from user where id={id}`
4. 三个角度：
   1. 通用性：全量属性更好
   2. 占用空间：部分属性更好
   3. 代码维护：表面上全量属性更好

### 缓存穿透优化

![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200318161322.png)

- 原因：

  1. 业务代码自身问题
  2. 恶意攻击、爬虫等等

- 发现

  1. 业务的相应时间
  2. 业务本身问题
  3. 相关指标：总调用数、缓存层命中数、存储层命中数

- 解决

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200318164417.png)

- 问题：

  1. 需要更多的键
  2. 缓存层和存储层数据“短期”不一致

```java
public String getPassThrough(String key){
    String cacheValue = cache.get(key);
    if(StringUtils.isBlink(cacheValue)){
        String storageValue = storage.get(key);
        cache.set(key,storageValue);
        //如果存储数据为空，需要设置一个过期时间（300秒）
        if(StringUtils.isBlink(storageValue)){
            cache.expire(key,60*5);
        }
        return storageValue;
    }else{
        return cacheValue;
    }
}
```

- ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200318165903.png)

### 无底洞问题优化

- 问题：
  - 2010年，Facebook有了3000个Memcache节点
  - 发现问题：“加”机器性能没能提升，反而下降
- ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200318174658.png)
- 优化IO的几种方法
  1. 命令本身优化：例如慢查询keys、hgetall bigkey
  2. 减少网络通信次数
  3. 降低接入成本：例如客户端长连接/连接池、NIO等

### 缓存雪崩优化

- 机器部署
  1. 机器添加部署脚本：ssh账号、redis安装部署
  2. [Cachecloud](https://github.com/sohutv/cachecloud)添加机器

### 热点key重建优化

- ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200318175519.png)

- 三个目标

  - 减少重缓存的次数
  - 数据尽可能一致
  - 减少潜在危险

- 两个解决

  - 互斥锁（mutex key）

    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200318175933.png)

      ```java
      String get(String key){
          String value = redis.get(key);
          if(value == null){
              String mutexKey = "mutex:key:"+key;
              if(redis.set(mutexKey,"1","ex 180","nx")){
                  value = db.get(key);
                  redis.set(key,value);
                  redis.delete(mutexKey);
              }else{
                  //其他线程休息50毫秒后重试
                  Thread.sleep(50);
                  get(key);
              }
          }
          return value;
      }
      ```

  - 永远不过期

    1. 缓存层面：没有设置过期时间（没有用expire）

    2. 功能层面：为每个value添加逻辑过期时间，但发现超过逻辑过期时间后，会使用单独的线程去构建缓存

       ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200318180917.png)

       ```java
       String get(final String key){
           V v = redis.get(key);
           String value = redis.get(key);
           long logicTimeout = v.getLogicTimeout();
           if(logicTimeout >= System.currentTimeMillis()){
               String mutexKey = "mutex:key:"+key;
               if(redis.set(mutexKey,"1","ex 180","nx")){
                   //异步更新后台异常执行
                   threadPool.execute(new Runnable(){
                       public void run(){
                           String dbValue = db.get(key);
                   		redis.set(key,(dbValue,newLogicTimeout));
                   		redis.delete(keyMutex);
                       }
                   });
               }
           }
           return value;
       }
       ```

  - 对比

    |   方案   |          优点           |                       缺点                       |
    | :------: | :---------------------: | :----------------------------------------------: |
    |  互斥锁  |  思路简单  保证一致性   |          代码复杂度增加  存在死锁的风险          |
    | 永不过期 | 基本杜绝热点key重建问题 | 不保证一致性  逻辑过期时间增加维护成本和内存成本 |

### 总结

- 缓存收益：加速读写、降低后端存储负载
- 缓存成本：缓存和存储数据不一致性、代码维护成本、运维成本
- 推荐结合剔除、超时、主动更新三种方案共同完成
- 穿透问题：使用缓存空对象和布隆过滤器来解决，注意它们各自的使用场景和局限性
- 无底洞问题：分布式缓存中，有更多的机器不保证有更高的性能。有四种批量操作方式：串行命令、串行IO、并行IO、hash_tag

## 十一、Redis云平台CacheCloud

### Redis规模化运维

- 问题
  - 发布构建繁琐，私搭乱盖
  - 节点&机器等运维成本
  - 监控报警初级
- [CacheCloud](https://github.com/sohutv/cachecloud)
  1. 一键开启Redis（Standalone、Sentinel、Cluster）
  2. 机器、应用、实例监控和报警
  3. 客户端：透明使用、性能上报
  4. 可视化运维：配置、扩容、Failover、机器/应用/实例上下线
  5. 已存在Redis直接接入和数据迁移
- 场景使用
  1. 全量视频缓存（视频播放API）：跨机房高可用
  2. 消息队列同步（RedisMQ中间件）
  3. 分布式布隆过滤器（百万QPS）
  4. 计数系统：计数（播放数）
  5. 其他：排行榜、社交（直播）、实时计算（反作弊）等

### 机器部署

1. 机器添加部署脚本：ssh账号、Redis安装部署
2. Cachecloud添加机器

### 应用接入

![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200320115804.png)

![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200320120059.png)

## 十二、小结

1. Redis初识：单机安装部署（版本选择），边界（使用场景）
2. API理解和使用：单线程、5种数据结构使用和选择
3. Redis客户端使用：Jedis、redis-py等，客户端很“简单”
4. Redis其他功能：慢查询、pipeline、发布订阅、bitmap等
5. redis持久化：RDB和AOF的优缺点和最佳实践
6. redis复制：配置方法、全量和部分复制、常见运维问题
7. redis sentinel：高可用、架构、“新”的客户端、基本原理
8. redis cluster：数据分布、架构、安装部署、扩容、客户端、常见问题
9. 缓存设计与优化：粒度、更新策略、无底洞问题、穿透、雪崩、热点key
10. cachecloud：平台化redis开发运维工具

## 十三、Redis布隆过滤器

### 引出布隆过滤器

- 问题：现有50亿个电话号，现有10万个电话号码，要快速准确判断这些电话号是否已经存在
  1. 通过数据库查询：实现快速有点难
  2. 数据预防在集合中：50亿*8字节≈40GB（内存浪费或不够）
  3. hyperloglog：准确有点难
- 类似问题：
  - 垃圾邮件过滤
  - 文件处理软件（例如Word）错误单词检测
  - 网络爬虫重复URL检测
  - Hbase行过滤

### 布隆过滤器原理

- 1970年伯顿.布隆提出，用很小的空间，解决上述类似问题
- 实现原理：一个很长的二进制向量和若干个哈希函数
- 构建：
  - 参数：m个二进制向量，n个预备数据，k个hash函数
  - 构建布隆过滤器：n个预备数据走一遍上面的过程
  - 判断元素存在：走一遍上面的过程：如果都是1，则表明存在，反之不存在

### 布隆过滤器误差率

- 肯定存在误差：恰好都命中了

- 直观因素：m/n的比率，hash函数的个数

- 实际误差率公式

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200325164844.png)

  - m/n与误差率成反比，k与误差率成反比

### 本地布隆过滤器

- 现有库：[guava](http://ifeve.com/google-guava-hashing/)
- 本地布隆过滤器的问题：
  1. 容量受限制
  2. 多个应用存在多个布隆过滤器，构建同步复杂

### Redis单机布隆过滤器

- ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200325171234.png)
- 实现方法
  - 定义布隆过滤器构造参数：m、n、k、误差概率
  - 定义布隆过滤器操作函数：add和contain
  - 封装Redis位图操作
  - 开发测试样例

### Redis分布式布隆过滤器

- 实现方法：
  - 多个布隆过滤器：二次路由
  - 基于pipeline提高效率

## 十四、Redis开发规范

### Key名设计

![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200327170152.png)

![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200327171733.png)

### Value设计

- 拒绝bigkey

  - 强制：

    - string类型控制在10kb以内
    - hash、list、set、zset元素个数不要超过5000
    - 反例：一个包含几百万个元素的list、hash等，一个巨大的json字符串

  - bigkey危害

    - 网络阻塞
    - redis阻塞
      - 慢查询：hgetall、lrange、zrange
    - 集群节点数据不均衡
    - 频繁序列化：应用服务器CPU消耗
      1. redis客户端本身不负责序列化
      2. 应用频繁序列化和反序列化bigkey：本地缓存或Redis缓存

  - bigkey发现

    - 应用异常

      ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200406111455.png)

    - redis-cli --bigkeys  `redis-cli -h your.ip -p xxx --bigkeys`

    - scan + debug object

    - 主动报警：网络流量监控、客户端监控

    - 内核热点key问题优化

  - bigkey删除

    1. 阻塞：注意隐性删除（过期、rename等）

       删除hash、list、set、sorted set四种数据结构不同数量不同元素大小的耗时

       |  key类型   | 10万（8个字节） | 100万（8个字节） | 10万（16个字节） | 100万（16个字节） | 10万（128个字节） | 100万（128个字节） |
       | :--------: | :-------------: | :--------------: | :--------------: | :---------------: | :---------------: | ------------------ |
       |    hash    |      51ms       |      950ms       |       58ms       |       970ms       |       96ms        | 2000ms             |
       |    list    |      23ms       |      134ms       |       23ms       |       138ms       |       23ms        | 266ms              |
       |    set     |      44ms       |      873ms       |       58ms       |       881ms       |       73ms        | 1319ms             |
       | sorted set |      51ms       |      845ms       |       57ms       |       859ms       |       59ms        | 969ms              |

    2. redis4.0：lazy delete（unlink命令）

       ```java
       // bigkey 删除方法
       public void delBigHash(String host, int port, String password, String bigHashKey) {
               Jedis jedis = new Jedis(host, port);
               if (password != null && "".equals(password)) {
                   jedis.auth(password);
               }
               ScanParams scanParams = new ScanParams().count(100);
               String cursor = "0";
               do {
                   ScanResult<Map.Entry<String, String>> scanResult = jedis.hscan(bigHashKey, cursor, scanParams);
                   List<Map.Entry<String, String>> entryList = scanResult.getResult();
                   if (entryList != null && !entryList.isEmpty()) {
                       for (Map.Entry<String, String> entry : entryList) {
                           jedis.hdel(bigHashKey,entry.getKey());
                       }
                   }
                   //cursor = scanResult.getStringCursor();  估计是个老方法  没找到
                   cursor = scanResult.getCursor();
               } while (!"0".equals(cursor));
       
               //删除bigkey
               jedis.del(bigHashKey);
           }
       ```

    3. bigkey预防

       - 优化数据结构：例如二级拆分
       - 物理隔离或者万兆网卡：不是治标方案
       - 命令优化：例如hgetall ->hmget、hscan
       - 报警和定期优化

  - bigkey总结

    - 牢记Redis单线程特性
    - 选择合理的数据结构和命令
    - 清楚自身OPS
    - 了解bigkey的危害

- 选择合适的数据结构

  - 例如：实体类型（数据结构内存优化：例如ziplist，注意内存和性能的平衡）

    - 反例：`set user:1:name tom; set user:1:age 19; set user:1:favor football;`
    - 正例：`hmset user:1 name tom age 19 favor football`

  - 需求：picId =>userId(100万)

    - 方案：

      - 全部string：set picId userId

      - 一个hash：hset allPics picId userId

      - 若干个小hash：hset picId/100 pic%100 userId

        ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200406165541.png)

      - 三种方案内存分析

        - 配置（支持动态修改）
          - hash-max-ziplist-entries 512
          - hash-max-ziplist-value 64
        - ziplist
          - 连续内存
          - 读写有指针位移，最坏O(n²)
          - 新增删除有内存重分配

      - 对比

        |   方案   |   优点   |                 缺点                  |
        | :------: | :------: | :-----------------------------------: |
        | 全String | 编程简单 |      浪费内存  全量获取较为复杂       |
        |  全hash  |   暂无   |           浪费内存  bigkey            |
        | 分段hash | 节省内存 | 编程复杂  超时问题  性能问题  ziplist |

- 过期设计

  - 键值生命周期（redis不是垃圾桶）
    - 周期数据需要设置过期时间，object idle time可以找垃圾key-value
    - 过期时间不宜集中：缓存穿透和雪崩等问题

### 命令使用技巧

1. 【推荐】O(n)以上命令关注N的数量

   - 例如：hgetall、lrange、smembers、zrange、sinter等并非不能使用，但需要明确N的值。有遍历的需求可以使用hscan、sscan、zscan代替。

2. 【推荐】禁用命令

   - 禁止线上使用keys、flushall、flushdb等，通过redis的rename机制禁掉命令，或者使用scan的方式渐进式处理

3. 【推荐】合理使用select

   - redis的多数据库较弱，使用数字进行区分
   - 很多客户端支持较差
   - 同时多业务用多数据库实际还是单线程处理，会有干扰

4. 【推荐】redis事务功能较弱，不建议过多使用

   - redis的事务功能较弱（不支持回滚）
   - 而且集群版本（自研和官方）要求一次事务操作的key必须在一个slot上（可以使用hashtag功能解决）

5. 【推荐】redis集群版本在使用Lua上有特殊要求

   - 所有key，必须在1个slot上，否则直接返回error
   - **"-ERR eval/evalsha command keys must in same slot\r\n"**

6. 【建议】必要情况下使用monitor命令时，要注意不要长时间使用

   ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200414141528.png)

### Java客户端优化

1. 【推荐】

   - 避免多个应用使用一个Redis实例
   - 正例：不相干的业务拆分，公共数据做服务化

2. 【推荐】

   - 使用连接池，标准使用方式

     ```java
     //redisHost为实例的IP，redisPort为实例端口，redisPassword为实例的密码，timeout既是连接超时又是读写超时
     JedisPool jedisPool = new JedisPool(jedisPoolConfig, redisHost, redisPort, timeout, redisPasswor//d);
     //执命令如下
     Jedis jedis = null;
     try {
         jedis = jedisPool.getResource();
         //具体的命令
         jedis.executeCommand()
     } catch (Exception e) {
         logger.error(e.getMessage(), e);
     } finally {
         //在JedisPool模式下，Jedis会被归还给资源池
         if (jedis != null) 
             jedis.close();
     }
     ```

     | 序号 |            参数名             |                             含义                             |       默认值       |                           使用建议                           |
     | :--: | :---------------------------: | :----------------------------------------------------------: | :----------------: | :----------------------------------------------------------: |
     |  1   |         testWhileIdle         |                     是否开启空闲资源检测                     |       false        |                             true                             |
     |  2   | timeBetweenEvictionRunsMillis |               空闲资源的检测周期（单位为毫秒）               |     -1：不检测     | 建议设置，周期自行选样，也可以默认，也可以使用JedisPoolConfig中的配置 |
     |  3   |  minEvictableIdleTimeMillis   | 资源池中资源最小空闲时间（单位为毫秒），达到此值后空闲资源将被移除 | 1000 60 30 =30分钟 | 可根据自身业务决定，大部分默认即可，也可以考虑使用JedisPoolConfig中的配置 |
     |  4   |    numTestsPerEvictionRun     |                做空闲资源检测时，每次的采样数                |         3          | 可根据自身应用连接数进行微调，如果设置为-1，就是对所有连接做空闲监测 |

     **其中JedisPoolConfig对一些参数的默认设置如下：**

     **testWhileIdle=true**

     **minEvictableIdleTimeMills=60000**

     **timeBetweenEvictionRunsMillis=30000**·

     **numTestsPerEvictionRun=-1**

   - 如何预估最大连接池
     - maxTotal怎么设置？maxIdle接近MaxTotal即可
       1. 考虑因素
          - 业务希望Redis并发量
          - 客户端执行命令时间
          - Redis资源：例如node（应用个数）*maxTotal不能超过Redis最大连接数
          - 资源开销：例如虽然希望控制空闲连接，但是不希望因为连接池的频繁释放创建连接造成不必要开销
       2. 例子
          - 一次命令时间（borrow|return resource + Jedis执行命令（含网络））的平均耗时约为1ms，一个连接的QPS大约是1000
          - 业务期望的QPS是50000
          - 理论的maxTotal = 50000/1000 = 50 个，可以适当伸缩

## 十五、内存管理

### 内存消耗

- 内存使用统计（进入redis-cli 直接输入 info）

  | 属性名                      | 属性说明                                                  |
  | :-------------------------- | :-------------------------------------------------------- |
  | **userd_memory**            | **Redis分配器分配的内存量，也就是实际存储数据的自存总量** |
  | used_memory_human           | 以可读格式返回Redis使用的内存总量                         |
  | **used_memory_rss**         | **从操作系统的角度，Redis进程占用的总物理内存**           |
  | used_memory_peak            | 内存分配器分配的最大内存，代表used_memory的历史峰值       |
  | used_memory_peak_human      | 以可读的格式显示内存消耗峰值                              |
  | used_memory_lua             | Lua引擎所消耗的内存                                       |
  | **mem_fragmentation_ratio** | **used_memory_rss/used_memory比值，表示内存碎片率**       |
  | mem_allocator               | Redis所使用的内存分配器。默认：jemalloc                   |

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200416101537.png)

- 内存消耗划分

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200416105846.png)

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200416110647.png)

### 内存缓冲

- 缓冲内存-客户端缓冲区

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200416112157.png)

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200416112353.png)

  - 普通客户端

    1. 默认：`client-output-buffer-limit normal 0 0 0`

    2. 默认：没有限制客户端缓冲

    3. 注意：防止大的命令或者monitor

       ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200416112914.png)

  - slave客户端

    1. 默认：`client-output-buffer-limit slave 256mb 64mb 60`
    2. 阻塞：主从延迟较高，或者从节点过多
    3. 注意：主从网络，从节点不要超过2个

  - pubsub客户端

    1. 默认：``client-output-buffer-limit pubsub 32mb 8mb 60``
    2. 阻塞：生产大于消费
    3. 注意：根据实际场景适当调试

- 缓冲内存-复制缓冲区

  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200416134820.png)
  - 缓冲内存-AOF缓冲区
    - 注意：AOF重写期间，AOF的缓冲区，没有容量限制
    - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200416135534.png)

- 对象内存
  1. key：不要过长，量大不容忽视（Redis3：embstr 39字节）
  2. value：ziplist、intsetde等优化方式
- 内存碎片
  1. 必然存在：jemalloc
  2. 优化方式：
     - 避免频繁更新操作：append、setrange等
     - 安全重启，例如redis sentinel和redis cluster等
- 子进程内存消耗
  1. 必然存在：fork（bgsave和bgrewriteaof）
  2. 优化方式：
     - 去掉THP特性：2.6.38增加的特性
     - 观察写入量：copy-on-write
     - overcommit_memory=1

### 内存管理

- 设置内存上限
  
  - 注意：定义实例最大内存，便于管理机器内存，一般要预留30%
  
    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200420154633.png)
  
    ```shell
    redis > config set maxmemory 6GB
    redis > config set maxmemory 2GB
    redis > config rewrite
    ```
  
- 动态调整内存上限

- 内存回收策略

  - 删除过期键值

    1. 惰性删除：访问key -> expired dict -> del key

    2. 定时删除：每秒运行10次，采样删除

       ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200420155255.png)

  - 内存溢出控制策略

    - 超过maxmemory后触发相应策略，由maxmemory-policy控制
      - Noeviction：默认策略，不会删除任何数据，拒绝所有写入操作并返回端错误信息“（error） OOM command not allowed when userd memory” 此时Redis只响应读操作由maxmemory-policy控制
      - Volatile-lru：根据LRU算法删除设置了超时属性（expire）的键，直到腾出足够空间为止。如果没有可删除的键对象，回退到noeviction策略
      - Allkeys-lru：根据LRU算法删除键，不管数据有没有设置超时属性，直到腾出足够空间为止
      - Allkeys-random：随机删除所有键，直到腾出足够空间为止
      - volatile-random：随机删除过期键，直到腾出足够空间为止
      - volatile-ttl：根据键值对象的ttl属性，删除最近将要过期数据。如果没有，回退到noeviction策略

### 内存优化

- 内存分布

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200416110647.png)

- 合理选择优化数据结构

  - 需求：计算网站每天独立用户数
  - 选择：集合、bitmaps、hyperloglog
  - ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200420172448.png)

- 客户端内存优化

  - 结构
    - 输入缓冲区：最大1G
    - 输出缓冲区，支持按类型设置：
      1. 普通客户端
      2. 发布订阅客户端
      3. 复制客户端
  - 一次内存暴增（原因）
    1. 批量写入
    2. 主从不一致
  - 处理和预防
    1. 处理：找到对应的业务方直接干掉
    2. 预防：
       1. 运维层面：线上Redis禁用monitor（rename-command）
       2. 运维层面：适度限制缓冲区大小
       3. 开发层面：理解monitor的原理，也可以短暂寻找热点key（本地执行）
       4. 开发层面：使用CacheCloud可以直接监控

- 其他方法

  1. 不要忽视key长度：一亿个键，一个字节也是节省。（键名：简短明了）`user:friends:notify:{id}----->u:f:n:{id}`
  2. 序列化和压缩方法：拒绝Java原生，采用portobuf、kryo、snappy等

- 需不需要用Redis？

  1. 数据：大数据、冷数据
  2. 功能性：关系型查询、消息队列

### 总结

1. 内存是宝贵资源
2. 结合场景选择和优化数据结构
3. 序列化时有成本的
4. 不要忽视键长度

## 十六、开发运维那些坑

### Linux内核优化

- vm.overcommit_memory

  - |  值  |                             含义                             |
    | :--: | :----------------------------------------------------------: |
    |  0   | 表示内核将检查是否有足够的可用内存。如果有足够的可用内存，内存申请通过，否则内存申请失败，并把错误返回给应用进程 |
    |  1   |             表示内核允许超量使用内存直到用完为止             |
    |  2   | 表示内核绝不过量（“never overcommit”）使用内存，即系统整个内存地址空间不能超过swap+50%的RAM值，50%时overcommit_ratio默认值，此参数同样支持修改 |

  - 获取和设置

    - 获取：

      `# cat /proc/sys/vm/overcommit_memory   0 `

    - 设置：

      `echo "vm.overcommit_memory=1" >> /etc/sysctl.conf`

      `sysctl vm.overcommit_memory=1`

  - 最佳实践

    1. Redis设置合理的maxmemory，保证机器有20%~30%的闲置内存
    2. 集中化管理AOF重写和RDB的bgsave
    3. 设置vm.overcommit_memory=1,防止极端情况下会造成fork失败

- swappiness

  - |  值  |                             策略                             |
    | :--: | :----------------------------------------------------------: |
    |  0   | Linux3.5及以上：宁愿用OOM killer 也不用swap    Linux3.4及更早：宁愿用swap也不用OOM killer |
    |  1   |          Linux3.5及以上：宁愿用swap也不用OOM killer          |
    |  60  |                            默认值                            |
    | 100  |                   操作系统会主动地使用swap                   |

  - 设置

    1. 立即生效：`echo {bestvalue} > /proc/sys/vm/swappiness`
    2. 永久生效：`echo vm.swappiness={bestvalue}>>/etc/sysctl.conf`

  - 最佳实践

    ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200420184749.png)

- THP(Transparent huge page)

  - 作用

    1. 作用：加速fork

    2. 建议：禁用，可能产生更大的内存开销

    3. 设置方法：`echo never>/sys/kernel/mm/transparent_hugepage/enabled`

    4. 坑：源码中是绝对路径，注意不同发行版本的区别

       ```php
       FILE*fp = fopen("/sys/kernel/mm/transparent_hugepage/enabled","r");
       if(!fb) return 0;
       ```

- OOM killer

  - 该机制会监控那些占用内存过大，尤其是瞬间占用内存很快的进程，然后防止内存耗尽而自动把该进程杀掉。内核检测到系统内存不足、挑选并杀掉某个进程的过程可以参考内核源代码linux/mm/oom_kill.c，当系统内存不足的时候，out_of_memory()被触发，然后调用select_bad_process()选择一个”bad”进程杀掉。如何判断和选择一个”bad进程呢？linux选择”bad”进程是通过调用oom_badness()，挑选的算法和想法都很简单很朴实：最bad的那个进程就是那个最占用内存的进程

- [NTP(Net Time Protocol)](https://blog.csdn.net/iloli/article/details/6431757)

- [ulimit](https://blog.csdn.net/skiwnc/article/details/84100095)

- [TCP backlog](https://www.cnblogs.com/Orgliny/p/5780796.html)

### 安全的Redis

- 全球crackit攻击

  - 2015年11月，全球35000+个redis主机受到攻击
  - 被攻击的redis特征
    1. Redis所在的机器有外网IP
    2. Redis以默认端口6379为启动端口，并且是对外网开放的
    3. Redis是以root用户启动的
    4. Redis没有设置密码
    5. Redis的bind设置为0.0.0.0或者""

- Redis安全七法

  1. 设置密码
     1. 服务端配置：requirepass和masterauth
     2. 客户端连接：auth命令和-a参数
     3. 相关建议：
        1. 密码要足够复杂，防止暴力破解
        2. masterauth不要忘记
        3. auth还是通过明文传输
  2. 伪装危险命令
     1. 服务端配置：rename-command为空或者随机字符
     2. 客户端连接：不可用或者使用指定随机字符
     3. 相关建议：
        1. 不支持config set动态设置
        2. RDB和AOF如果包含rename-command之前的命令，将无法使用
        3. config命令本身是在Redis内核会使用到，不建议设置

  3. bind
     1. 服务端配置：bind限制的是网卡，并不是客户端ip
     2. 相关建议：
        1. bind不支持config set
        2. bind 127.0.0.1需要谨慎
        3. 如果存在外网网卡尽量屏蔽掉
  4. 防火墙：杀手锏
  5. 定期备份
  6. 不使用默认端口，防止被弱攻击杀掉
  7. 使用非root用户启动

### 热点key发现

- 客户端

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200421100545.png)

- 代理端

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200421100624.png)

- 服务端

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200421100655.png)

- 机器收集

  ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/20200421100750.png)

### 总结

|  方案  |                     优点                     |                             缺点                             |
| :----: | :------------------------------------------: | :----------------------------------------------------------: |
| 客户端 |                   实现简单                   |     1. 内存泄漏隐患 2. 维护成本高 3. 只能统计单个客户端      |
|  代理  | 代理是客户端和服务端的桥梁，实现最方便最系统 |                   增加代理端的开发部署成本                   |
| 服务端 |                   实现简单                   | 1. monitor本身的使用成本和危害，只能短时间使用 2. 只能统计单个redis节点 |
|  机器  |        对于客户端和服务端无侵入和影响        |       需要专业的运维团队开发，并且增加了机器的部署成本       |

