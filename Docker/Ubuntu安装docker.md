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
3. 安装VMware Workstation Pro 激活密钥: `ZF3R0-FHED2-M80TY-8QYGC-NPKYF`  安装镜像，此步骤如果不会请[百度](https://www.baidu.com/s?ie=UTF-8&wd=vmware workstation pro 如何安装ubuntu镜像)

## 二、修改Ubuntu相关配置

**[相关配置文件下载 清华软件源、MySQL配置文件、redis配置文件、docker镜像源配置文件](https://zelen.lanzous.com/ivBGiop8tgh)**

**切换软件源为[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)根据自己的安装版本选择**

```shell
# 简单来说就是：apt = apt-get、apt-cache 和 apt-config 中最常用命令选项的集合
# 1.安装lrzsz工具，方便windows和linux间的文件上传下载
sudo apt update
sudo apt lrzsz
# 2.备份镜像源 参考 2
sudo cp -v /etc/apt/sources.list /etc/apt/sources.list.backup
# 3.更换镜像源
sudo rm -ef /etc/apt/sources.list
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
# 安装方法挺多的 我是直接按照官方方式安装的 下列命令的相关解释请查看 参考 4
# 我是新环境无需执行卸载步骤 如果需要的自行卸载
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

## docker安装相关镜像及配置

```shell
# 1. 创建相关文件 zne为我新创建的文件夹存放我的安装相关数据 此路径为之后安装镜像的相关配置路径 可按照个人习惯创建
sudo mkdir -pv /zne/portainer/data
sudo mkdir -pv /zne/mysql/{conf,data,logs} 
sudo mkdir -pv /zne/redis/{conf,data,logs}
# 将下载的配置文件放置到对应的文件夹下 my.cnf --> /zne/mysql/conf  redis.conf --> /zne/redis/conf

# 2. 下载相关镜像
docker pull portainer/portainer
docker pull mysql
docekr pull redis

# 3. 运行镜像
# Portainer 参考 6 直接浏览器 运行 Ubuntu IP:9000 创建新的用户即可
docker run -p 9000:9000 -p 8000:8000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /zne/portainer/data:/data -d portainer/portainer
# MySQL
docker run --name mysql --restart=always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime -v /zne/mysql/data:/var/lib/mysql -v /zne/mysql/conf:/etc/mysql/conf.d -v /zne/mysql/logs:/logs -d mysql
# 修改MySQL 密码为root 若想用123456的则不必执行下列命令
# 进入容器
docker exec -it mysql8 bash
mysql -uroot -p
123456
# 修改MySQL8为简易密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
ALTER USER 'root'@'%' IDENTIFIED BY 'root';
# Redis
docker run -it -v /zne/redis/data:/var/lib/redis -v /zne/redis/conf/redis.conf:/etc/redis/redis.conf -v /zne/redis/logs:/logs --restart=always --name redis -p 6379:6379 -d redis redis-server /etc/redis/redis.conf --appendonly yes
```

## 相关参考

1. [ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)
2. [ubuntu20.04更改国内镜像源](https://blog.csdn.net/weixin_42118522/article/details/106069030)
3. [Ubuntu 中apt update和upgrade 的区别](https://blog.csdn.net/zhjulia123/article/details/83479515)
4. [Install Docker Engine on Ubuntu | Docker Documentation](https://docs.docker.com/engine/install/ubuntu/)
5. [ubuntu查看和修改时区](https://blog.csdn.net/cau_eric/article/details/90478798)
6. [Docker 图形化工具 Portainer](https://mp.weixin.qq.com/s/YRqISK4yJo9J9WzzTvD9CQ)
