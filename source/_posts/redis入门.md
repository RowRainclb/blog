---
title: redis入门
date: 2017-09-12 13:35:46
tags:
      - redis
---
# 前言
Redis是常用基于内存的Key-Value数据库，比Memcache更先进，支持多种数据结构，高效，快速。用Redis可以很轻松解决高并发的数据访问问题；做为时时监控信号处理也非常不错。
# 安装
```javascript
//在终端中安装Redis服务器端
sudo apt-get install redis-server
```
安装完成后，Redis服务器会自动启动，我们检查Redis服务器程序
```javascript
//在终端中检查Redis服务器系统进程
ps -aux|grep redis
```
```javascript
//在终端中通过启动命令检查Redis服务器状态
netstat -nlt|grep 6379
```
```javascript
//通过启动命令检查Redis服务器状态
sudo /etc/init.d/redis-server status
```
# 通过命令行客户端访问Redis
安装Redis服务器，会自动地一起安装Redis命令行客户端程序。

在本机输入redis-cli命令就可以启动，客户端程序访问Redis服务器
```javascript
~ redis-cli
redis 127.0.0.1:6379>

# 命令行的帮助
redis 127.0.0.1:6379> help
redis-cli 2.2.12
Type: "help @" to get a list of commands in 
      "help " for help on 
      "help " to get a list of possible help topics
      "quit" to exit


# 查看所有的key列表
redis 127.0.0.1:6379> keys *
(empty list or set)
```
## 基本的Redis客户端命令操作
增加一条字符串记录key1
```javascript
# 增加一条记录key1
redis 127.0.0.1:6379> set key1 "hello"
OK

# 打印记录
redis 127.0.0.1:6379> get key1
"hello"
```
增加一条数字记录key2
```javascript
# 增加一条数字记录key2
set key2 1
OK

# 让数字自增
redis 127.0.0.1:6379> INCR key2
(integer) 2
redis 127.0.0.1:6379> INCR key2
(integer) 3

# 打印记录
redis 127.0.0.1:6379> get key2
"3"
```
增加一条列表记录key3
```javascript
# 增加一个列表记录key3
redis 127.0.0.1:6379> LPUSH key3 a
(integer) 1

# 从左边插入列表
redis 127.0.0.1:6379> LPUSH key3 b
(integer) 2

# 从右边插入列表
redis 127.0.0.1:6379> RPUSH key3 c
(integer) 3

# 打印列表记录，按从左到右的顺序
redis 127.0.0.1:6379> LRANGE key3 0 3
1) "c"
2) "b"
3) "a"
```
增加一条哈希表记录key4
```javascript
# 增加一个哈希记表录key4
127.0.0.1:6379> HSET key4 name "clb"
(integer) 1
# 在哈希表中插入，email的Key和Value的值
127.0.0.1:6379> HSET key4 email "xxx@gmai.com"
(integer) 1
# 打印哈希表中，name为key的值
127.0.0.1:6379> HGET key4 name
"clb"
# 打印整个哈希表
127.0.0.1:6379> HGETALL key4
1) "name"
2) "clb"
3) "email"
4) "xxx@gmai.com"
```
.增加一条哈希表记录key5
```javascript
# 增加一条哈希表记录key5，一次插入多个Key和value的值
127.0.0.1:6379> HMSET key5 username clb pwd 123456 age 26
OK
# 打印哈希表中，username和age为key的值
127.0.0.1:6379> HMGET key5 username age
1) "clb"
2) "26"
# 打印完整的哈希表记录
127.0.0.1:6379> HGETALL key5
1) "username"
2) "clb"
3) "pwd"
4) "123456"
5) "age"
6) "26"
```

删除记录
```javascrpt
# 查看所有的key列表
redis 127.0.0.1:6379> keys *

# 删除key1,key5
redis 127.0.0.1:6379> del key1
(integer) 1
redis 127.0.0.1:6379> del key5
(integer) 1

# 查看所有的key列表
redis 127.0.0.1:6379> keys *
1) "key2"
2) "key3"
3) "key4"
```

# 修改Redis的配置
##  使用Redis的访问账号
默认情况下，访问Redis服务器是不需要密码的，为了增加安全性我们需要设置Redis服务器的访问密码。设置访问密码为redis。

用vi打开Redis服务器的配置文件redis.conf
```javascript
~ sudo vi /etc/redis/redis.conf

#取消注释requirepass
requirepass foobared
```
## 让Redis服务器被远程访问
默认情况下，Redis服务器不允许远程访问，只允许本机访问，所以我们需要设置打开远程访问的功能。

用vi打开Redis服务器的配置文件redis.conf
```javascript
~ sudo vi /etc/redis/redis.conf

#注释bind
#bind 127.0.0.1
```
修改后，重启Redis服务器。
```javascript
~ sudo /etc/init.d/redis-server restart
Stopping redis-server: redis-server.
Starting redis-server: redis-server.
```
未使用密码登陆Redis服务器
```javascript
~ redis-cli

redis 127.0.0.1:6379> keys *
(error) ERR operation not permitted
```
发现可以登陆，但无法执行命令了。

登陆Redis服务器，输入密码
~  redis-cli -a foobared

redis 127.0.0.1:6379> keys *
1) "key2"
2) "key3"
3) "key4"
登陆后，一切正常。

我们检查Redis的网络监听端口
```javascript
//检查Redis服务器占用端口
~ netstat -nlt|grep 6379
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN
```
我们看到从之间的网络监听从 127.0.0.1:6379 变成 0 0.0.0.0:6379，表示Redis已经允许远程登陆访问

我们在远程的另一台Linux访问Redis服务器
```javascript
~ redis-cli -a foobared -h 192.168.x.xxx

redis 192.168..x.xxx:6379> keys *
1) "key2"
2) "key3"
3) "key4"
```
远程访问正常。通过上面的操作，我们就把Redis数据库服务器，在Linux Ubuntu中的系统安装完成

[原文地址](http://blog.fens.me/linux-redis-install/)

推荐文章:
异步社区 : [学习Redis从这里开始](http://www.epubit.com.cn/article/200)
在线测试: [在线测试](http://try.redis.io/)