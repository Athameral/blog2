---
title: mysql_phpadmin_docker
date: 2024-09-07 16:33:28
tags: [MySQL, Docker]
---

# 通过Docker配置MySQL和phpMyAdmin

> 以下笔记均以Windows为操作系统，但GNU/Linux也可以参考。
> 
> 我相信用GNU/Linux的你有更好的动手能力。

## Step 1. 安装Docker Desktop

前往[Docker官网](https://www.docker.com/products/docker-desktop/)下载，保持默认设置安装即可。

执行`docker --version`来检查安装，你应该能看到类似的输出：

    C:\Users\XXX>docker --version
    Docker version 25.0.3, build 4debf41

在进行下一步操作前，请确保Docker Desktop处在运行状态。

## Step 2. 拉取MySQL和phpMyAdmin的镜像（Images）

执行下面的命令：

    docker pull mysql
    docker pull phpmyadmin

> 你也可以使用Docker Desktop提供的GUI来操作。

## Step 3. 用镜像构建容器（Containers）并运行（run）

    docker run --name BDDE_mysql -d -e MYSQL_ROOT_PASSWORD=change_me -e MYSQL_TCP_PORT=50123 -p 50123:50123
    docker run --name BDDE_mysql -d --link BDDE_mysql:db -e PMA_ARBITRARY=1 -p 8080:80

访问[http://127.0.0.1:8080](http://127.0.0.1:8080)即可进入phpMyAdmin界面。

数据库地址输入`db:50123`，用户名为`root`，密码为`change_me`。

> 端口和密码请自行修改。

## Step 4. 使用Docker Volume来保存MySQL容器中的数据

- 创建一个`volume`：

        docker volume create BDDE_mysql_data

- 将`Step 3`中的第一行命令替换为：

        docker run --name BDDE_mysql -d -e MYSQL_ROOT_PASSWORD=change_me -e MYSQL_TCP_PORT=50123 -p 50123:50123 -v BDDE_mysql_data:/var/lib/mysql/
