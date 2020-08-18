---
date: 2020-08-18 09:39:41
categories: 
 - Docker
 - MySQL8
tags: 
 - Docker
 - MySQL8
---

# MySQL8在win10 docker上的相关安装

```shell
docker pull mysql
# 挂载本地磁盘路径需手动创建 随系统自动启动
docker run -it -v E:/Docker/docker/mysql/data:/var/lib/mysql -v E:/Docker/docker/mysql/config/my.cnf:/etc/mysql/my.cnf --restart=always --name mysql8 -e MYSQL_ROOT_PASSWORD=system123 -p 3307:3306 -d mysql
# 不随系统自动启动
docker run -it --rm  -v E:/Docker/docker/mysql/data:/var/lib/mysql -v E:/Docker/docker/mysql/config/my.cnf:/etc/mysql/my.cnf --name mysql80 -e MYSQL_ROOT_PASSWORD=system123456 -p 3308:3306 -d mysql
```

[挂载文件](https://zelen.lanzous.com/i9dxgfqvq6j)  [相关参考](https://my.oschina.net/summergao/blog/3066063)

![](https://gitee.com/zelen/IMG/raw/master/PicGo/20200818094507.png)

**将挂载文件所在目录开放给docker**

**进入容器**

`docker exec -it mysql8 bash`

**修改MySQL8为简易密码**

` ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';`

` ALTER USER 'root'@'%' IDENTIFIED BY 'root';`

[参考](https://www.cnblogs.com/c-x-a/p/12632122.html)

