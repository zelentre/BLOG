---
title: Win10 WSL 安装 docker 相关流程
date: 2021-03-28 20:47:55
categories: 
 - wsl
 - docker
tags: 
 - Win10 WSL 安装 docker 相关流程
---

# Win10 WSL 安装 docker 相关流程

<!-- more -->

## 一、相关下载及配置

1. [下载docker desktop](https://www.docker.com/products/docker-desktop)

2. 微软商或者[网站](https://docs.microsoft.com/en-us/windows/wsl/install-manual#installing-your-distro)下载相应的win10 子系统

3. [开启 wsl2](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10)

4. 下载 [LxRunOffline-vx.x.0-msvc](https://github.com/DDoSolitary/LxRunOffline/releases/tag/v3.5.0) 此步骤为迁移子系统到D盘，若C盘足够可忽略此步骤

   - 在LxRunOffline下载目录按住SHIFT并右键鼠标，选择“在此处打开Powershell窗口”，进入界面后输入.\LxRunOffline.exe list即可查询目前本机有的子系统以及位置

     ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/image-20210328205752655.png)

   - 子系统迁移  **使用`lxrunoffline move`进行迁移 ， -n 指定你要迁移的系统名 ，-d 指定你新系统的迁移路径**

     ![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/image-20210328210100012.png)

   - 使用`LxRunOffline.exe get-dir`查询系统目录，查看是否迁移成功  `LxRunOffline.exe get-dir -n 子系统名称`

   - [参考链接](https://zhuanlan.zhihu.com/p/145753804)

5. 安装 docker，若C盘足够同样忽略一下 步骤

   - 看图![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/image-20210328211606211.png)
   - 镜像加速 [阿里云](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors) 容器镜像服务-->镜像工具-->镜像加速器-->加速地址![](https://gcore.jsdelivr.net/gh/znej/pic/picgo/image-20210328211652150.png)将docker数据从C盘迁移出去

   - 关闭docker

   - 关闭所有发行版 `wsl --shutdown`

   - 将docker-desktop-data、docker-desktop导出 

     ```shell
     wsl --export docker-desktop-data D:\wsl\docker-desktop-data\docker-desktop-data.tar
     wsl --export docker-desktop D:\wsl\docker-desktop\docker-desktop.tar
     ```
   
   - 注销docker-desktop-data 

     ````shell
     wsl --unregister docker-desktop-data
     wsl --unregister docker-desktop
     ````
   
   - 重新导入docker-desktop-data

     ```shell
     wsl --import docker-desktop-data D:\wsl\docker-desktop-data\ D:\wsl\docker-desktop-data\docker-desktop-data.tar --version 2
     wsl --import docker-desktop D:\wsl\docker-desktop\ D:\wsl\docker-desktop\docker-desktop.tar --version 2
     ```
   
   - 重启docker

## 二、docker 镜像下载及启动（以Ubuntu为例）

### 一、将配置文件挂载到Ubuntu

**访问Ubuntu文件 win+ r 在输入 \\wsl$\Ubuntu-20.04\  注意替换自己的子系统名**

1. 创建 相关文件

   ```shell
   sudo mkdir -pv /usr/local/mysql/{conf,data,logs} 
   sudo mkdir -pv /usr/local/redis/{conf,data,logs} 
   ```

2. 下载镜像

   ```shell
   # 不指定版本 默认下载最新版本镜像
   docker pull mysql
   docker pull redis
   ```

3. 镜像启动

   1. MySQL 配置文件[下载](https://zelen.lanzous.com/imB5Bnevqih)将下载的文件放入到 `/usr/local/mysql/conf`也可自己重新指定文件路径

      ```shell
      docker run --name mysql --restart=always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime -v /usr/local/mysql/data:/var/lib/mysql -v /usr/local/mysql/conf:/etc/mysql/conf.d -v /usr/local/mysql/logs:/logs -d mysql
      ```

   2. Redis配置文件[下载 ](https://zelen.lanzous.com/iektOil70vc) 文件放入路径`/usr/local/redis/conf/redis.conf`

      ```shell
      docker run -it -v /usr/local/redis/data:/var/lib/redis -v /usr/local/redis/conf/redis.conf:/etc/redis/redis.conf -v /usr/local/redis/logs:/logs --restart=always --name redis -p 6379:6379 -d redis redis-server /etc/redis/redis.conf --appendonly yes
      ```

### 二、配置文件在docker容器中

**若要修改配置文件需要到指定的容器中进行修改**

1. 镜像启动

   ```shell
   # MySQL /usr/local/mysql/conf为空文件
   docker run --name mysql --restart=always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime -v /usr/local/mysql/data:/var/lib/mysql -v /usr/local/mysql/conf:/etc/mysql/conf.d -v /usr/local/mysql/logs:/logs -d mysql
   # Redis /usr/local/redis/conf为空文件
   docker run -it -v /usr/local/redis/data:/var/lib/redis -v /usr/local/redis/conf:/etc/redis/redis.conf -v /usr/local/redis/logs:/logs --restart=always --name redis -p 6379:6379 -d redis redis-server /etc/redis/redis.conf --appendonly yes
   ```

### 三、其他相关命令

**修改配置文件、下载配置文件等操作**

```shell
# docker cp :用于容器与主机之间的数据拷贝。
# 6bb94da8196e为容器id 可通过 docker ps 查看
docker cp 6bb94da8196e:/etc/mysql/ D:\ZNE\1\
docker cp D:\ZNE\1\ 6bb94da8196e:/etc/mysql/
# 修改容器配置文件
# 进入指定容器
docker exec -it mysql bash
# 查看 linux版本
cat /etc/issue
# 更新 apt-get 若无法更新 尝试更换配置 /etc/apt/sources.list
apt-get update
# 安装 vim
apt-get install vim -y
# 配置文件路径 在启动镜像时已指定 也可通过 find 查找
```

[Linux find 命令](https://www.runoob.com/linux/linux-comm-find.html)