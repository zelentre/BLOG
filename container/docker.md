**简介**
 &nbsp;&nbsp;&nbsp;&nbsp;Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化,容器是完全使用沙箱机制，相互之间不会有任何接口。

<!--more-->

一个完整的Docker有以下几个部分组成：

  1. DockerClient客户端
  2. Docker Daemon守护进程
  3. Docker Image镜像
   4. DockerContainer容器
   5. [一些docker命令](https://blog.csdn.net/qq_38503329/article/details/97147797)

**docker 安装 mysql（其他类似）**

1. 拉取镜像  指定自己需要的版本，后续启动容器时也需要指定版本（最新版默认不需要）

   ` docker pull mysql:5.7`

2. 运行容器  -p 3306:3306 指定ip、指定宿主机port、指定容器port ip不指定 默认0.0.0.0

   ` docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql:5.7`

3. 其他后续 包括修改配置文件

   `dockerexec -it f7bbac /bin/bash `  // f7bbac 是容器ID docker ps 可查

   ```shell
   apt-get update
   apt-get install vim -y
   cd /etc/mysql/mysql.conf.d
   vim mysqld.cnf
   lower_case_table_names=1 //末尾添加 是MySQL不区分大小写 重启后生效
   show variables like '%table_names';  //查看是否区分大小写
   ```

**docker安装最新版MySQL（后两条 修改密码格式 方便 Navicat 连接 MySQL8 以上的版本  本地` 'root'@'localhost'`  远程  `'root'@'%'`）**

```shell
docker pull mysql
docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql
dockerexec -it f7bbac /bin/bash
select host, user, authentication_string, plugin from user;
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
```