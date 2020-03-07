# Redis.md

## Redis初始

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

   - ```linux
     ## 创建redis的软连接，方便以后升级
     ln -s redis-3.0.7 redis
     ```

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

     - ```linux
       ps -ef | grep redis
       netstat -antpl | grep redis
       redis-cli -h ip -p port ping
       redis-cli -h 127.0.0.1 -p 6379
       ```

   - 动态参数启动

     - ```linux
       redis-server --port 6380 ## 端口随意指定
       ```

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

## API的理解和使用

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

    ```
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

    ```
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

    ```
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

    ```
    127.0.0.1:6379> hmset user:2:info age 30 name kaka page 50
    OK
    127.0.0.1:6379> hlen user:2:info
    (integer) 3
    127.0.0.1:6379> hmget user:2:info age name
    1) "30"
    2) "kaka"
    ```

  - ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305112446.png)

    ```
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

    ```
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

  - ```
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

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305174633.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305171233.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305171320.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305171437.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305172209.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305172314.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305172429.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305174255.png)

    ![img](https://gitee.com/zelentre/IMG/raw/master/PicGo/20200305174339.png)

    ```
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

## Redis客户端的使用

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

  + ```java
    // 1.String
    // 输出结果：ok
    jedis.set("hello","world");
    // 输出结果：world
    jedis.get("hello");
    // 输出结果：1
    jedis.incr("counter");
    ```

  + ```java
    // 2.hash
    jedis.hset("myhash","f1","v1");
    jedis.hset("myhash","f2","v2")；
    // 输出结果：{f1=v1, f2=v2}
    jedis.hgetall("myhash");
    ```

  + ```java
    // 3.list
    jedis.rpush("mylist","1");
    jedis.rpush("mylist","2");
    jedis.rpush("mylist","3");
    // 输出结果：[1,2,3]
    jedis.lrange("mylist",0,-1);
    ```

  + ```java
    // 4.set
    jedis.sadd("myset","a");
    jedis.sadd("myset","b");
    jedis.sadd("myset","a");
    // 输出结果：[b, a]
    jedis.smembers("myset");
    ```

  + ```java
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

      + ```java
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