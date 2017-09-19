---
title: 'docker的mysql8镜像,数据库乱码问题'
date: 2017-09-19 12:55:12
tags:
       - docker
       - mysql
---
# docker的mysql镜像乱码问题解决办法
最近使用docker构建mysql镜像时,数据库中数据出现乱码,记录一下解决方法
基础镜像使用daocloud.io/library/mysql:8

在容器内进入mysql,查看编码show varables like “%char%”；
发现 default-character-set     default-character-set   character-set-server 默认都是latain, 并不支持中文

修改方法:
1 创建文件 utf8mb4.cnf,这个就是sql的配置文件，作用是把默认字符集改为utf8mb4
内容如下:
```javascript
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

2 把utf8mb4.cnf放在Dockerfile 同一目录下
3 修改Dockerfile,基于mysql 官方的docker镜像，把utf8mb4.cnf 复制到容器的/etc/mysql/conf.d/目录下，构建新镜像
修改如下:
```javascript
#基础镜像使用daocloud.io/library/mysql:8
FROM daocloud.io/library/mysql:8
# 设置mysql默认编码,防止中文乱码出现
COPY utf8mb4.cnf /etc/mysql/conf.d/
```
4 构建新镜像
```javascript
docker build -t mysql:0.1.0 .
```
5 运行docker即可
docker run --name mysql -idt mysql:0.1.0

再次查看数据库,编码正常
