---
title: docker构建nginx1.13
date: 2018-05-08 14:58:33
tags:
       - docker
       - nginx
---
# docker构建nginx镜像  

概述

基于http://blog.csdn.net/u011186019/article/details/53712253安装好docker

使用nginx镜像在docker中运
nginx1.13.0

一、pull nginx镜像

使用网易docker镜像：https://c.163.com/hub#/m/repository/?repoId=2967

运行：docker pull hub.c.163.com/library/nginx:latest

pull完成使用docker images命令查看

二、启动nginx容器

运行命令：docker run -p 8080:80 --name nginx_web -it hub.c.163.com/library/nginx /bin/bash

该命令是将容器的nginx的80端口映射成系统8080端口，并进入容器命令界面

启动nginx：nginx
退出容器：Ctrl+P+Q

三、访问nginx
http://172.16.67.1:8080/

成功

