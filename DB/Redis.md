

<!-- more -->

[TOC]

----https://www.bilibili.com/video/av94180858/

## 一、Redis初始

### redis特性
1. 速度快

   ![内存](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200303174922.png)

   - 10W OPS（每秒可以实现10万次读写 官方数字）
   - 数据存在内存中
   - 用C语言编程（50000 line）
   - 单线程模型

2. 持久化（断电不丢数据）

   - Redis所有数据保存在内存中，对数据的更新将异步地保存到磁盘上

3. 多种数据结构

   ![数据结构](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200303180006.png)

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
   - ![缓存](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304092708.png) 当redis中有data时直接返回data，当redis中无data-->storage,storage-->appserver、storage-->redis,以便下次访问直接走redis
2. 计数器
   - ![计数](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304093255.png)转发数、评论数用缓存计数，还有视频播放数等等
3. 消息队列系统
4. 排行榜
5. 社交网络
   - ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304093712.png) 粉丝数、共同关注数。。
6. 实时系统
   - 垃圾邮件系统

### redis安装

1. redis安装

   - ![安装](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304094433.png)
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

   - ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304100658.png)

5. redis客户端返回值

   - ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304100959.png)
   - ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304101031.png)

### redis常用配置

- daemonize ---> 是否是守护进程（no|yes）
- port ---> redis对外端口号（默认6379）
- logfile ---> redis系统日志
- dir ---> redis工作目录

## 二、API的理解和使用

### 通用命令

- 通用命令

  - keys（redis里的所有的键）![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304105306.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304173232.png)

  -  **keys命令一般不再生产环境使用** **keys\*怎么用 1.热备从节点 2.scan**

  - dbsize（计算数据库大小） 

    ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304105824.png)

  - exists key（键是否存在） 

  ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304110033.png)

  - del key [key ...]（删除key，可以多个） 

    ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304110127.png)

  - expire key seconds（设置过期时间） ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304110526.png)

    ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304110712.png)

    ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304110829.png)

  - type key（key类型） ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304111007.png)![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304111055.png)

  - **时间复杂度大部分是O(1)**

- 数据结构和内部编码

  ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304112932.png)

  ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304140640.png)

- 单线程架构 ![redis同一时间只会执行一个命令](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304141413.png)**单线程快的原因**

  - 纯内存
  - 非阻塞IO![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304141813.png)
  - 避免线程切换和竞态消耗 ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304142116.png)

### 字符串类型

- 结构和命令

  - ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304142451.png)**建议在100KB以内**

  - ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304143253.png)![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304143346.png)

  - ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200304143927.png)![img](https://gitee.com//zelentre/IMG/raw/master/PicGo/20200304151748.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304170522.png)![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304170853.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304171046.png)![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304172437.png)

     ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304171152.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304171244.png)

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
  - 实现：分布式id生成器 ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304162534.png)![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304162814.png)

- 内部编码

- 查漏补缺

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304171507.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304171550.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304171732.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304171823.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304171858.png)

### 哈希类型

- 特点

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304175609.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304175636.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304175926.png)

- 重要API

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304180239.png)

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

  - ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200305103327.png)

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

  - ![img](https://raw.githubusercontent.com/zelentre/IMG/master/PicGo/20200305103100.png)

    ```shell
    127.0.0.1:6379> hmset user:2:info age 30 name kaka page 50
    OK
    127.0.0.1:6379> hlen user:2:info
    (integer) 3
    127.0.0.1:6379> hmget user:2:info age name
    1) "30"
    2) "kaka"
    ```

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305112446.png)

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

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305114023.png)

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

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305135110.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305135214.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305135306.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305135333.png)

- hash vs string

- 查漏补缺

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305135928.png)
  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305135957.png)

### 列表类型

- 特点

  - **有序、可以重复、左右两边插入弹出**

- 重要API

  - **增**![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305140613.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305140650.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305140841.png)

  - **删**![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305141347.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305141420.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305141755.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305141650.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305141914.png)

  - **查**![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305142058.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305142200.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305142257.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305142400.png)

  - **改**![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305142501.png)

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

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305143135.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305143523.png)

- 查漏补缺

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305143628.png)
  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305143723.png)

### 集合类型

- 特点

  - **不能插入重复元素，` srem key music`删除元素**![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305144302.png)
  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305144821.png)
  - **无序、无重复、集合间操作**

- 集合内API

  - **S开头**

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305150055.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305164445.png)

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

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305170048.png)

- 集合间API

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305170220.png)
  - 实现：共同关注（sinter）
  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305170414.png)

### 有序集合类型

- 特点

  - 结构![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305170705.png)
  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305170743.png)
  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305170840.png)

- 重要API

  - **以Z开头**

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305174533.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305174633.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305171233.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305171320.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305171437.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305172209.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305172314.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305172429.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305174255.png)

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305174339.png)

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

  - 实现：排行榜![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305173811.png)

- 查漏补缺

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305173929.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305173949.png)

## 三、Redis客户端的使用

### Java客户端：Jedis

+ 获取jedis

  + shell（redis-cli）  Java（jedis）

    ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200306103814.png)

  + ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200306103935.png)

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

    + ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200306113555.png)

  + Jedis连接池

    + ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200306113844.png)

  + 方案对比

    + ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200306134849.png)

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

    + ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200306141709.png)
    + ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200306142536.png)
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
      - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200306144528.png)
    + 解决思路
      - 慢查询阻塞：池子连接都被hang住。
      - 资源池参数不合理：例如QPS高、池子小。
      - 连接泄密（没有close()）：此类问题比较难定位，例如client list、netstat等，最重要的是代码。
      - DNS异常等。

## 四、Redis其他功能

### 慢查询

1. 生命周期

   - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309095118.png)

2. 两个配置

   ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309095543.png)

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

  - **一次网路命令通信模型**![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304171244.png)

  - **批量网络命令通信模型**![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200304171152.png)
  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309103325.png)
  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309103738.png)

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

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309111739.png)
  - **订阅这可以订阅多个频道**

- API

  - publish

    - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309112125.png)

  - subscribe

    - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309112203.png)

  - unsubscribe

    - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309112241.png)

  - 其他

    ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309112346.png)

- 发布订阅与消息队列

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309112429.png)
  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309112608.png)

### Bitmap

- 位图

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309113011.png)

- 相关命令

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309143205.png)
  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309143831.png)
  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309143926.png)
  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309144012.png)
  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309144120.png)

- 独立用户统计

  1. 使用set和Bitmap
  2. 1亿用户，5千万独立

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309144549.png)
  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309144651.png)
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

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309151432.png)

- 使用经验

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309151656.png)

### GEO

- GEO是什么

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309151922.png)
  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309152004.png)

- 5个城市经纬度

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309152029.png)

- 相关命令

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309152309.png)

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309152422.png)

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309152513.png)

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309152538.png)

    ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309152646.png)

- 相关说明

  1. since 3.2+
  2. type geoKey = zset
  3. 没有删除API：zrem key member

## 五、Redis持久化的取舍和选择

### 持久化的作用

- 什么是持久化
  - redis所有数据保持在内存中，对数据的更新将异步地保存到磁盘上
- 持久化的实现方式
  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309154132.png)

### RDB

- 什么是RDB

  - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309154714.png)

- 触发机制-主要三种方式

  - save（同步）

    - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309155136.png)
    - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309155204.png)
    - ![](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200309155246.png)

  - bgsave（异步）

    - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309164147.png)
    - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309164246.png)
    - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309164333.png)
    - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309164353.png)

  - 自动

    - **自动生成RDB**

      ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309165036.png)

    - ```shell
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

    - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309165940.png)

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
    - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309173446.png)
  - 不可控、丢失数据
    - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309173612.png)
- 什么是AOF
  - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309173720.png)
  - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309173807.png)
- AOF三种策略
  - always
    - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309174020.png)
  - everysec
    - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309174135.png)
  - no
    - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309174219.png)
  - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309174304.png)
- AOF重写
  - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309174657.png)
  - 作用：
    - 减少硬盘占用量
    - 加速恢复速度
  - AOF重写实现两种方式
    - bgrewriteaof
      - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309175535.png)
    - AOF重写配置
      - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309175607.png)
      - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309180145.png)
      - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309180238.png)
      - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309180304.png)
      - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200309180622.png)

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

- ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310114122.png)
- AOF阻塞定位
  - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310134905.png)
  - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310134922.png)
  - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310134939.png)

## 七、Redis复制的原理与优化

### 什么是主从复制

- 单机问题：

  - 机器故障
  - 容量瓶颈
  - QPS瓶颈

- 主从复制的作用

  - **一主一从**

    ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310140307.png)

  - **一主多从**

    ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310140451.png)

  - **作用**

    - 数据副本
    - 扩展读性能

  - **总结**

    1. 一个master可以有多个slave
    2. 一个slave只能有一个master
    3. 数据流向是单向的，master到slave

### 复制的配置

- slaveof命令

  - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310141243.png)
  - ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310141458.png)

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

  ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310152018.png)

- **全量复制开销**

  1. bgsave时间
  2. RDB文件网络传输时间
  3. 从节点清空数据时间
  4. 从节点加载RDB的时间
  5. 可能的AOF重写时间

- **部分复制**

  ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310160012.png)

### 故障处理

- **主从结构-故障转移**

  - slave宕掉

    ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310160740.png)

  - master宕掉

    ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310160902.png)

### 开发运维常见问题

1. 读写分离

   ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310161840.png)

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

      ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310173149.png)

      - 问题：主节点重启，多从节点复制
      - 解决：更换复制拓扑

   2. 单机器复制风暴

      ![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200310173236.png)

      - 机器宕机后，大量全量复制
      - 主节点分散多机器