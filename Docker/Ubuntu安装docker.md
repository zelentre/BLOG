---
title: Ubuntu安装docker
date: 2021-05-01 22:01:08
categories: 
 - docker
tags: 
 - Ubuntu
 - docker
---

# Ubuntu安装docker

<!--more--> 

## 一、准备工作下载相软件、镜像

1. [下载win10 VMware Workstation Pro](https://www.vmware.com/go/getworkstation-win)如果想下载其他版本请[前往](https://www.vmware.com/products/workstation-pro/workstation-pro-evaluation.html)
2. [下载Ubuntu服务器版镜像](https://releases.ubuntu.com/20.04.2/ubuntu-20.04.2-live-server-amd64.iso)如果想下载其他版本请[前往](https://cn.ubuntu.com/download)
3. 安装VMware Workstation Pro 激活密钥: `ZF3R0-FHED2-M80TY-8QYGC-NPKYF`  安装镜像，此步骤如果不会请[百度](https://www.baidu.com/s?ie=UTF-8&wd=vmwareworkstationpro如何安装ubuntu镜像)

## 二、修改Ubuntu相关配置

**[相关配置文件下载 清华软件源、MySQL配置文件、redis配置文件、elasticsearch配置文件、kibana配置文件、docker镜像源配置文件](https://zelen.lanzoui.com/ihS85tjde5a)**

**切换软件源为[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)根据自己的安装版本选择**

```shell
# 简单来说就是：apt = apt-get、apt-cache 和 apt-config 中最常用命令选项的集合
# 1.安装lrzsz工具，方便windows和linux间的文件上传下载
sudo apt update
sudo apt install lrzsz
# 2.备份镜像源 参考 2
sudo cp -v /etc/apt/sources.list /etc/apt/sources.list.backup
# 3.更换镜像源
sudo rm -rf /etc/apt/sources.list
# 或者直接重命名
sudo mv /etc/apt/sources.list /etc/apt/sources.list.backup
# 上传已经编辑好的sources.list到 /etc/apt/ 目录下 或者使用vim编辑sources.list(不推荐 太慢了)
# 4.更新 参考 3
# apt update：只检查，不更新（已安装的软件包是否有可用的更新，给出汇总报告）
sudo apt update 
# apt upgrade：更新已安装的软件包具体可百度自行查找
sudo apt upgrade 软件包名(不填则更新所有)
# 5.修改Ubuntu时区 参考 5
# 查看时间
date -R
# 修改时区
timedatectl set-timezone "Asia/Shanghai"
```

## 三、安装docker

```shell
# 安装方法挺多的 我是直接按照官方方式安装的 可以根据脚本方式安装Install using the convenience script 下列命令的相关解释请查看 参考 4
# 我是新环境无需执行卸载步骤 如果需要的自行卸载

# 方法一 推荐（脚本方式安装）
curl -fsSL https://get.docker.com -o get-docker.sh

sudo sh get-docker.sh
# 方法二
sudo apt update

sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io

# 查看是否安装成功
sudo docker

# 安装成功在进行以下步骤，若未成功请自查原因 版本 Ubuntu Server 20.04.2 LTS

# 修改docker镜像源 按照下载的daemon.json配置 直接vim 我的是没有daemon.json这个配置文件的vim帮我自动创建了，若你有此文件 则进行修改 wq保存退出
sudo vim /etc/docker/daemon.json

{"registry-mirrors":[
	"https://registry.docker-cn.com",
	"https://docker.mirrors.ustc.edu.cn"
]}

# 将当前用户 添加 Docker用户组 zne 用户名 尝试过重启docker但没起作用 因此直接 reboot
sudo groupadd docker.service
sudo usermod -aG docker zne
sudo reboot
```

## 四、docker安装相关镜像及配置

```shell
# 1. 创建相关文件 可按照个人习惯创建
sudo mkdir -pv /usr/local/portainer/data
sudo mkdir -pv /usr/local/mysql/{conf,data,logs} 
sudo mkdir -pv /usr/local/redis/{conf,data,logs}
# 将下载的配置文件放置到对应的文件夹下 my.cnf --> /usr/local/mysql/conf  redis.conf --> /usr/local/redis/conf

# 2. 下载相关镜像
# portainer/poratainer 已弃用 https://hub.docker.com/r/portainer/portainer
docker pull portainer/portainer
# Portainer 2.0 的所有新版本都将在 portainer/portainer-ce 中发布 推荐使用下面这个
docker pull portainer/portainer-ce
docker pull mysql
docker pull redis

# 3. 运行镜像
# Portainer 参考 6 直接浏览器 运行 Ubuntu IP:9000 创建新的用户即可 废弃
docker run -p 9000:9000 -p 8000:8000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /usr/local/portainer/data:/data -d portainer/portainer
# Portainer-ce 参考 6 直接浏览器 运行 Ubuntu IP:9000 创建新的用户即可
docker run -p 9000:9000 -p 8000:8000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /usr/local/portainer/data:/data -d portainer/portainer-ce
# MySQL
docker run --name mysql --restart=always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime -v /usr/local/mysql/data:/var/lib/mysql -v /usr/local/mysql/conf:/etc/mysql/conf.d -v /usr/local/mysql/logs:/logs -d mysql
# 修改MySQL 密码为root 若想用123456的则不必执行下列命令
# 进入容器
docker exec -it mysql bash
mysql -uroot -p
123456
# 修改MySQL8为简易密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
ALTER USER 'root'@'%' IDENTIFIED BY 'root';
# Redis
docker run -it -v /usr/local/redis/data:/var/lib/redis -v /usr/local/redis/conf/redis.conf:/etc/redis/redis.conf -v /usr/local/redis/logs:/logs --restart=always --name redis -p 6379:6379 -d redis redis-server /etc/redis/redis.conf --appendonly yes
```

### 安装rabbitmq

```shell
# https://hub.docker.com/_/rabbitmq 查找rabbitmq镜像
# 此方法需要手动开启它的管理后台(我使用的这种)
docker pull rabbitmq
# 拉取带web管理界面的
docker pull rabbitmq:3.8.19-management
# 带图形化界面（推荐使用这个）
docker pull rabbitmq:management
# 创建相关挂载目录
sudo mkdir -pv /usr/local/rabbitmq/{conf,data,logs}
# 图像化镜像启动
docker run -p 15672:15672 -p 5672:5672 --name rabbitmq --restart=always -v /usr/local/rabbitmq/data:/var/lib/rabbitmq/mnesia -v /usr/local/rabbitmq/logs:/var/log/rabbitmq/log -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -d rabbitmq:management
# 启动镜像
docker run -p 15672:15672 -p 5672:5672 --name rabbitmq --restart=always -v /usr/local/rabbitmq/data:/var/lib/rabbitmq/mnesia -v /usr/local/rabbitmq/logs:/var/log/rabbitmq/log -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -d rabbitmq
# 进入容器并开启web管理界面
docker exec -it rabbitmq bash
rabbitmq-plugins enable rabbitmq_management
```

### 安装MongoDB

```shell
# 创建相关挂载目录
sudo mkdir -pv /usr/local/mongo/{conf,data,logs}
# 在 /usr/local/mongo/conf目录下创建mongo配置文件，内容在下面
touch mongodb.conf
# 获取镜像
docker pull mongo
# 启动镜像 参考 8
docker run --name mongo --restart=always -p 27017:27017 -v /usr/local/mongo/data:/data/db -v /usr/local/mongo/conf:/data/conf -v /usr/local/mongo/logs:/data/log -d mongo
```

```shell
#端口
port=27017
#数据库文件存放目录
dbpath=/data/mongo/data
#日志文件存放路径
logpath=/data/mongo/log
#使用追加方式写日志
logappend=true
#以守护线程的方式运行，创建服务器进程
fork=true
#最大同时连接数
maxConns=100
#不启用验证
#noauth=true
#每次写入会记录一条操作日志
journal=true
#存储引擎有mmapv1、wiredTiger、mongorocks
storageEngine=wiredTiger
#访问IP
bind_ip=0.0.0.0
#用户验证
#auth=true
```

### 安装MinIO

```shell
# 相关参考 9
# 创建相关挂载目录
sudo mkdir -pv /usr/local/minio/{conf,data,logs}
# 获取镜像
docker pull minio/minio
# 启动镜像
docker run -p 9090:9000 -p 9001:9001 --name minio --restart=always -v /usr/local/minio/data:/data -e MINIO_ROOT_USER=minioadmin -e MINIO_ROOT_PASSWORD=minioadmin -d minio/minio server /data --console-address ":9001"
```
### 安装elasticsearch

```shell
# 相关参考 10（主要） 11（辅助）
# 创建相关挂载目录
sudo mkdir -pv /usr/local/elk/elasticsearch/{config,data,logs,plugins}
#如果有需要，创建用户定义的网络（用于连接到连接到同一网络的其他服务（例如：Kibana））
docker network rm mynet
docker network create --driver bridge --subnet 172.18.0.0/16 --gateway 172.18.0.1 mynet
# 获取镜像 
docker pull elasticsearch:7.14.1
# 先最简启动一个elasticsearch 容器 复制他的相关配置文件
docker run --name elasticsearch -d -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 elasticsearch:7.14.1
# 复制配置文件
docker cp elasticsearch:/usr/share/elasticsearch/config /usr/local/elk/elasticsearch/
docker cp elasticsearch:/usr/share/elasticsearch/plugins /usr/local/elk/elasticsearch/
# 切换身份为root 先将原配置文件重命名，在将配置文件elasticsearch.yml复制到config下(直接在xshell中拖动即可，注意一定要用root)
mv elasticsearch.yml elasticsearch.yml.old
# 修改目录权限
sudo chmod -vR 777 /usr/local/elk/
# 删除容器并重新启动
# 启动镜像
docker run --name elasticsearch --restart=always --net mynet -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" -v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime -v /usr/local/elk/elasticsearch/config/:/usr/share/elasticsearch/config -v /usr/local/elk/elasticsearch/data:/usr/share/elasticsearch/data -v /usr/local/elk/elasticsearch/logs:/usr/share/elasticsearch/logs -v /usr/local/elk/elasticsearch/plugins:/usr/share/elasticsearch/plugins -d elasticsearch:7.14.1
```
### 安装kibana

```shell
# 相关参考 10、11 
# 创建相关挂载目录
sudo mkdir -pv /usr/local/elk/kibana/{config,data,logs,plugins}
# 获取镜像 
docker pull kibana:7.14.1
# 先最简启动一个kibana 容器 复制他的相关配置文件
docker run --name kibana -p 5601:5601 -v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime -d kibana:7.14.1
# 复制配置文件
docker cp kibana:/usr/share/kibana/config /usr/local/elk/kibana/
docker cp kibana:/usr/share/kibana/plugins /usr/local/elk/kibana/
# 切换身份为root 先将原配置文件重命名，在将配置文件kibana.yml复制到config下(直接在xshell中拖动即可，注意一定要用root)
mv kibana.yml kibana.yml.old
# 删除容器并重新启动
# 启动镜像
docker run --name kibana --restart=always --net mynet -p 5601:5601 -v /usr/local/elk/kibana/config:/usr/share/kibana/config -v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime -d kibana:7.14.1
```



### docker相关命令

```shell
# 开机启动
systemctl enable docker
# 启动docker
systemctl start docker
# 关闭
systemctl stop docker
# 找镜像 替换mysql
docker search mysql
# 下载镜像 此处为指定版本5.7 可直接mysql（下载最新版mysql）
docker pull mysql:5.7
# 删除镜像   f6509bac4980为IMAGE ID
docker rmi f6509bac4980
# 查看容器列表  -a 查看所有容器
docker ps -a
# 停止运行的容器  使用容器名或者容器id
docker stop container-name/container-id
docker start container-name/container-id
# 删除容器
docker rm container-id
#容器日志
docker logs container-name/container-id
```
![docker](https://img-blog.csdnimg.cn/2020043011175696.png)

> -i：表示运行容器
> -d：在run后面加上-d参数,则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加-it两个参数，创建后就会自动进去容器）。
> -di：后台运行容器；
> –name：指定容器名；
> -p：指定服务运行的端口（5672：应用访问端口；15672：控制台Web端口号）；
> -v：映射目录或文件（文件共享），格式 宿主机目录：容器目录
> –hostname：主机名（RabbitMQ的一个重要注意事项是它根据所谓的 “节点名称” 存储数据，默认为主机名）；
> -e 指定环境变量；（RABBITMQ_DEFAULT_VHOST：默认虚拟机名；RABBITMQ_DEFAULT_USER：默认的用户名；RABBITMQ_DEFAULT_PASS：默认用户名的密码）

## 相关参考

1. [ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)
2. [ubuntu20.04更改国内镜像源](https://blog.csdn.net/weixin_42118522/article/details/106069030)
3. [Ubuntu 中apt update和upgrade 的区别](https://blog.csdn.net/zhjulia123/article/details/83479515)
4. [Install Docker Engine on Ubuntu | Docker Documentation](https://docs.docker.com/engine/install/ubuntu/)
5. [ubuntu查看和修改时区](https://blog.csdn.net/cau_eric/article/details/90478798)
6. [Docker 图形化工具 Portainer](https://mp.weixin.qq.com/s/YRqISK4yJo9J9WzzTvD9CQ)
7. [为什么用docker安装rabbitmq打不开管理页面呢](https://blog.csdn.net/weixin_44763595/article/details/109528165)
8. [docker安装mongo容器并挂载外部配置文件及目录](https://blog.csdn.net/weixin_45345374/article/details/116271582)
9. [Github标星28K+！MinIO这款可视化的对象存储服务真香！](https://mp.weixin.qq.com/s/qHjOEeQ3CaA0U4a2YBi3Pw)
10. [lejing-mall/run_elk_install_v7.13.2.sh](https://github.com/Weasley-J/lejing-mall/blob/main/shell/run_elk_install_v7.13.2.sh)
11. [docker安装elasticsearch（最详细版）](https://blog.csdn.net/qq_40942490/article/details/111594267)
12. [Install Kibana with Docker | Kibana Guide ](https://www.elastic.co/guide/en/kibana/7.14/docker.html)
13. [Docker的自定义网络](https://www.cnblogs.com/Kit-L/p/13251786.html)

