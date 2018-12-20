---
title: docker
date: 2018-12-20 11:28:16
author: 木子
top: true
categories: Docker
tags:
  - Docker
---
docker

Docker 是一个开源的应用容器引擎。

特点

Docker 是容器技术，从主机上隔离出多个独立的子系统，容器技术和主机是共享系统内核

虚拟机是从主机中完全隔离出的一个系统



由于Docker 与主机共享内核，有一下优势：

1、速度更快

Docker容器开启只需50毫秒

2、更加轻量

虚拟机中每个都是一套独立的系统，使得每个虚拟机都非常大。Docker 使用了分层的技术，不同的镜像之间可以共享相同的层，这使得容器尺寸很小

3、更加节省资源

Docker 是共享主机内核，启动一个docker就和启动一个普通程序一样，所以我们可以开启上千个Docker容器。

Docker化

将程序进行Docker化

什么是Docker化？

是指将开发的系统制作成一个镜像，当新的机器上要部署该项目时，只需要拉取镜像然后就可以直接创建容器来运行，这样就省去了重新安装运行环境的步骤

安装

Docker 可以安装、运行在常见的 Linux 系统上，而对于 Windows和 Mac 系统也有办法安装

windows：Windows10专业版，Windows10教育版，可以进行直接安装，如果是老的版本需要安装 Docker 虚拟机。

执行命名，查看是否安装成功

    docker -v  
    docker --version

如果想要查看 docker 的详细信息，执行

    docker version

使用docker





关键词

镜像（image）：已经打包好的 Docker 应用，类似于一个程序的安装包

镜像仓库： 存储镜像的服务器   

         连接地址：https://hub.docker.com/

容器：有一个镜像的时候我们可以创建多个容器，容器就是运行着的镜像，容器之间是隔离的



常用指令

  指令                  	说明                            
  docker images       	查看已经下载的镜像                     
  docker rmi 镜像名称：标签名 	删除已下载的镜像                      
  docker search 镜像    	从官方仓库（hub.docker.com）查找镜像     
  docker pull 镜像名称：标签名	从官方仓库下载镜像（标签名默认是latest） 代表最新版本
  docker run          	创建容器                          
  docker ps           	查看已经运行的容器                     
  docker ps -a        	查看所有创建的容器                     
  docker rm 容器名称      	删除已经停止的容器                     
  docker rm -f  容器名称  	删除正在运行的容器                     
  docker start 容器名称   	启动容器                          
  docker stop 容器名称    	停止容器                          
  docker  restart 容器名称	重启容器                          
  docker exec         	执行容器中的指令                      
                      	                              

创建容器

创建容器常用的参数：

    docker run --name 容器名称 -d -p 主机端口：容器内端口 -e 环境变量 --link 其他容器名：容器别名 镜像名称：标签名

- --name  指定容器的名称
- -d     容器在后台运行
- -p     绑定端口号，容器内部的端口号无法在外部访问，必须经过绑定之后才可以进行访问
- -e   设置容器中的环境变量
- --link    连接其他的容器

示例：创建一个mysql容器，密码是 123123，绑定班底 13306端口到容器的 3306端口

    docker run --name mysql001 -d -p 13306:3306 -e MYSQL_ROOT_PASSWORD=123123 mysql:5.6.42

- MYSQL_ROOT_PASSWORD   环境变量用来设置这个容器中的MySQL的密码

创建成功后，如果容器启动成功，使用docker ps  指令查看

 

	

启动之后，容器内部就运行了 mysql 服务器，在容器内部监听 3306 端口，已经将这个端口绑定到了主机的 13306 端口





进入容器

进入容器使用   docker exec  指令来实现

docker exec 指令：[执行一个运行着的容器中的命令]

    docker exec -it 容器名称 命令

- docker exec -it mysql001 mkdir data

进入容器内：[docker exec]

    docker exec -it mysql001 bash

或

    docker exec -it mysql001 /bin/sh

- -it    以实时交互的形式运行 （与 -d 正好相反）
  执行  exit指令  可以退出容器 

链接容器

Docker  推荐我们一个容器中只运行一个主要的应用程序

例如：我们要创建一个PHP + MySQL

- 运行一个 MySQL 容器

    docker run --name mysql001 -d -p 13306:3306 -e MYSQL_ROOT_PASSWORD mysql:latest

- 连接SQL库



- 创建 PHP 容器

    docker run --name php -d --link mysql001:mysql php:7.2.12-fpm-alpine3.8

- 创建链接之后就可以在容器中使用别名进行通信

    //  进入容器
    docker exec -it 容器名称 /bin/sh
    //  创建文件
    touch pdo.php
    //  进入文件进行编辑
    vi pdo.php
    
    //  内容
    $pdo = new PDO('mysql:host=链接别名;dbname=数据库名','root','123123')



扩展：我们可以同时链接多个容器

    docker run --name php001 -d --link mysql001:mysql1 --link mysql002:mysql2 php:7.2.12-fpm-alpine3.8



注：

     当创建容器的时候，如果发现本地没有镜像就会自动下载最新的镜像，下载完之后就创建容器

挂载硬盘

问题一、数据是保存在容器里的，如果容器删除了数据也就删除了。

 问题二、每次要修改容器时，必须要进入到容器中去修改





为了能够保存（持久化）数据以及共享容器间的数据，Docker 提出了 Volume 的概念。

可以使用 -v  这个参数，将容器中的一个目录或文件  和 主机上的目录或文件进行绑定，绑定之后，修改主机上的这个文件就相当于修改了容器中的文件，删除容器之后，绑定的目录和文件还在主机（不会被删除）。



为了实现主机和容器之间的数据共享，我们可以在创建容器时添加  -v  参数

    docker run .... -v 主机目录：容器中的目录  ....

示例：创建mysql 容器并将数据目录挂载到主机

    docker run --name mysql55 -d -v E:/mysql55:/var/lib/mysql -p 13308:3306 -e MYSQL_ROOT_PASSWORD=123123 mysql:5.6.42