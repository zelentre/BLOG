---
title: Redis在win10 docker上的相关安装
date: 2020-08-18 09:39:41
categories: 
 - Docker
 - Redis
tags: 
 - Docker
 - Redis
---

# Redis在win10 docker上的相关安装

```shell
docker pull redis
# 挂载本地磁盘路径需手动创建 随系统自动启动
docker run -it -v E:/Docker/docker/redis/data:/var/lib/redis -v E:/Docker/docker/redis/config/redis.conf:/etc/redis/redis.conf --restart=always --name redis -p 6379:6379 -d redis redis-server /etc/redis/redis.conf --appendonly yes
```

  [相关参考](https://www.cnblogs.com/chenlizhi/p/13654922.html)

![](https://testingcf.jsdelivr.net/gh/znej/pic/picgo/20220530130216.png)

[参考](https://blog.csdn.net/qq_37334135/article/details/77717248)

