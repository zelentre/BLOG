---
title: mongo在win10 docker上的相关安装
date: 2020-9-2 15:44:13
categories: 
 - Docker
 - mongo
tags: 
 - Docker
 - mongo
---

# mongo在win10 docker上的相关安装

<!--more-->

```shell
docker pull mongo
docker volume create --name mongodb
docker run -it -v mongodb:/data/db --name mongo -p 27017:27017 -d mongo
```

**简单创建**

[参考链接](http://www.moguf.com/post/windockerrunmongo)

**其他相关命令**

```shell
# 删除数据卷
docker volume rm mongodb
# 查看数据卷
docker volume ls
```

