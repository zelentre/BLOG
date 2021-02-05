---
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
   - 首先，需要打开“系统虚拟机平台”功能，在“控制面板\所有控制面板项\程序和功能”中选择“启用或者关闭Windows功能”，勾选对应选项即可：![image-20210205102305690](https://gitee.com/zelen/IMG/raw/master/PicGo/image-20210205102305690.png)

![image-20210205102336980](https://gitee.com/zelen/IMG/raw/master/PicGo/image-20210205102336980.png)

4. 打开Microsoft Store，下载Ubuntu20.04

5. cmd 查看 `wsl -l --v`

6. 查看linux目录 `\\wsl$\Ubuntu-20.04\`

   ![image-20210205102754173](https://gitee.com/zelen/IMG/raw/master/PicGo/image-20210205102754173.png)

7. docker 相关

   ![image-20210205103053072](https://gitee.com/zelen/IMG/raw/master/PicGo/image-20210205103053072.png)

   ![image-20210205103035157](https://gitee.com/zelen/IMG/raw/master/PicGo/image-20210205103035157.png)

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

10. 安装MySQL相关命令

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

11. 安装Redis相关命令

    ```shell
    docker pull redis
    # 挂载 随系统启动
    docker run -it -v /mnt/wsl/docker/redis/data:/var/lib/redis -v /mnt/wsl/docker/redis/config/redis.conf:/etc/redis/redis.conf --restart=always --name redis -p 6379:6379 -d redis redis-server /etc/redis/redis.conf --appendonly yes
    ```

    

