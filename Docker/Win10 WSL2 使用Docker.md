---
title: Win10 WSL2 使用Docker
date: 2021-02-05 10:14:33
categories: 
 - wsl
 - docker
tags: 
 - win10
 - wsl
 - docker
---

# Win10 WSL2 使用Docker

<!-- more -->

## 一、相关环境配置

1. win10环境为专业版（不是的自行百度找秘钥）
2. [安装docker-desktop](https://www.docker.com/products/docker-desktop)
3. 打开系统虚拟机平台
   - 首先，需要打开“系统虚拟机平台”功能，在“控制面板\所有控制面板项\程序和功能”中选择“启用或者关闭Windows功能”，勾选对应选项即可：![](https://fastly.jsdelivr.net/gh/znej/pic/picgo/20220530170140.png)

![](https://fastly.jsdelivr.net/gh/znej/pic/picgo/20220530170205.png)

4. 打开Microsoft Store，下载Ubuntu20.04

   - **将Ubuntu安装到D盘，步骤如下**
- [现在对应版本的Ubuntu](https://docs.microsoft.com/en-us/windows/wsl/install-manual#installing-your-distro)
- 将文件解压到你要指定的路径 解压
     - 双击 ubuntu2004.exe运行即可

5. cmd 查看 `wsl -l --v`  

   ![](https://fastly.jsdelivr.net/gh/znej/pic/picgo/20220530170227.png)

6. 查看linux目录 `\\wsl$\Ubuntu-20.04\`

   ![](https://fastly.jsdelivr.net/gh/znej/pic/picgo/20220530170246.png)

7. docker 相关

   ![](https://fastly.jsdelivr.net/gh/znej/pic/picgo/20220530170303.png)

   ![](https://fastly.jsdelivr.net/gh/znej/pic/picgo/20220530170332.png)

8. 将docker数据从C盘迁移出去

   - 关闭docker

   - 关闭所有发行版 `wsl --shutdown`

   - 将docker-desktop-data导出 ，docker-desktop占用空间小暂不处理（处理方式类似）

     ```shell
     wsl --export docker-desktop-data D:\wsl\docker-desktop-data\docker-desktop-data.tar
     ```

   - 注销docker-desktop-data `wsl --unregister docker-desktop-data`

   - 重新导入docker-desktop-data

     ```shell
     wsl --import docker-desktop-data D:\wsl\docker-desktop-data\ D:\wsl\docker-desktop-data\docker-desktop-data.tar --version 2
     ```

   - 重启docker

9. [docker 镜像相关挂载文件](https://zelen.lanzous.com/inqvqlc4msh)将文件放在 `/mnt/wsl/docker/`

## 二、安装相关镜像

### 安装MySQL相关命令

```shell
docker pull mysql
# 挂载 随系统启动
docker run -it -v /mnt/wsl/docker/mysql/data:/var/lib/mysql -v /mnt/wsl/docker/mysql/config/my.cnf:/etc/mysql/my.cnf --restart=always --name mysql8 -e MYSQL_ROOT_PASSWORD=system123456 -p 3306:3306 -d mysql
# 进入容器
docker exec -it mysql8 bash
mysql -uroot -p
system123456
# 修改MySQL8为简易密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
ALTER USER 'root'@'%' IDENTIFIED BY 'root';
```

### 安装Redis相关命令

```shell
docker pull redis
# 挂载 随系统启动
docker run -it -v /mnt/wsl/docker/redis/data:/var/lib/redis -v /mnt/wsl/docker/redis/config/redis.conf:/etc/redis/redis.conf --restart=always --name redis -p 6379:6379 -d redis redis-server /etc/redis/redis.conf --appendonly yes
```

## 相关参考

https://www.jianshu.com/p/a20c2d58eaac

https://zhuanlan.zhihu.com/p/148511634

https://docs.docker.com/docker-for-windows/wsl/

https://my.oschina.net/u/4313107/blog/4404650

https://www.jianshu.com/p/e79f4c938000