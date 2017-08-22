---
title: 在ubuntu14.04单机安装配置zookeeper和kafka
date: 2017-03-12 17:26:49
tags:
---
# ubuntu14.04单机安装配置zookee和kafka

为了方便以后扩展分布式的需要，运用Apache Kafka这个分布式消息发布订阅系统。Apache kafka的详细介绍详见官网
运行Apache Kafka，需要先安装好jdk和zookeeper。jdk安装过程就不赘述了。
## 1.安装配置zookeeper单机模式
这里，我们选择的是zookeeper-3.4.5这个版本，官网下载zookeeper-3.4.5。
下载之后将zookeeper-3.4.5.tar.gz移动到主文件夹并重命名：
1	mv Downloads/zookeeper-3.4.5.tar.gz zookeeper.tar.gz
然后解压缩为zookeeper文件夹：
2	tar -zxvf zookeeper.tar.gz
切换到zookeeper/conf目录：
3	cd /home/young/zookeeper/conf
复制zoo_simple.cfg为zoo.cfg:
4	cp zoo_simple.cfg zoo.cfg
打开zoo.cfg并修改内容如下：
```javascript
initLimit=10
syncLimit=5
dataDir=/home/young/zookeeper/data
clientPort=2181
```
配置好后，手动创建dataDir目录： mkdir /home/young/zookeeper/data
为zookeeper配置环境变量，打开/etc/profile，在结尾添加如下两句，并保存：
```javascript
  export ZOOKEEPER_HOME=/home/young/zookeeper
  export PATH=.:$ZOOKEEPER_HOME/bin:$JAVA_HOME/bin:$PATH
```
切换到zookeeper/bin目录： cd /home/young/zookeeper/bin
启动zookeeper的server：   ./zkServer.sh start
显示效果如下，则证明zookeeper配置成功：
 JMX enabled by default
 Using config: /home/young/zookeeper/bin/zookeeper/conf/zoo.cfg
 Starting zookeeper ... STARTED
需要结束服务时，仍在本目录下，执行如下命令即可： ./zkServer.sh stop
## 2.安装配置kafka单机模式
我们选择的是kafka_2.10-0.8.1.1.tgz，下载链接在这里：Apache kafka。下载之后放到主文件夹，并改名为kafka.tgz，然后解压缩到当前文件夹： tar -zxvf kafka.tgz
切换到kafka/config目录： cd /home/young/kafka/config
 这里，我们需要修改4个配置文件：
server.properties，zookeeper.properties，producer.properties，consumer.properties。
### 2.1 配置server.properties
下面几项是必须修改的，其他项目为默认配置：
```javascript
#broker.id需改成正整数，单机为1就好
broker.id=1
#指定端口号
port=9092
#localhost这一项还有其他要修改，详细见下面说明
host.name=localhost
#指定kafka的日志目录
log.dirs=/home/young/kafka/kafka-logs
#连接zookeeper配置项，这里指定的是单机，所以只需要配置localhost，若是实际生产环境，需要在这里添加其他ip地址和端口号
zookeeper.connect=localhost:2181
```
然后手动创建log.dirs空目录： mkdir /home/young/kafka/kafka-logs

### 2.2 配置zookeeper.properties

很简单，这个文件暂时只需要修改3项：
```javascript
#数据目录
dataDir=/home/young/kafka/zookeeper/data
#客户端端口
clientPort=2181
host.name=localhost
```
然后手动创建dataDir空目录： mkdir /home/young/kafka/zookeeper/data

### 2.3 配置producer.properties
只需要指定一项：
```javascript
metadata.broker.list=localhost:9092
```
### 2.4 配置consumer.properties
只需要指定一项：
```javascript
zookeeper.connect=localhost:2181
```
### 2.5 补充说明
对于这个版本的kafka，在配置好以上4项之后，仍然需要做2件事：
•	配置localhost，修改/etc/hosts文件
我的hosts文件里面，localhost部分是如下配置：
```javascript 
127.0.0.1   localhost
#下面这句，你的计算机名是什么就填什么，我的是young
127.0.0.1   young
255.255.255.255 broadcasthost
::1 localhost
fe80::1%lo0 localhost
```
	
这里如果不修改不添加，就会产生异常java.net.UnknownHostException。
添加slf4j-simple-1.7.2.jar
这里是个bug，/home/young/kafka/libs这个目录缺少slf4j-simple-1.7.2.jar这个文件，
只有slf4j-api-1.7.2.jar这个文件是不够的，必须两个都有。或者可以下载其他版本的两个slf4j文件，放入本目录。slf4j-simple-1.7.2.jar可以去官网下载slf4j-1.7.2.tar.gz，下载后解压缩，就可以看到这两个文件了。
如果这两个文件不全，就会有错误： 
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
•	
环境安装配置部分到此结束，下面介绍简单使用。

## 3.kafka的使用
### 3.1 启动zookeeper服务
切换至/home/young/kafka目录，执行以下命令，以启动zookeeper服务：
   bin/zookeeper-server-start.sh config/zookeeper.properties
### 3.2 启动kafka服务
仍在/home/young/kafka目录下，执行以下命令，以启动kafka服务：
   bin/kafka-server-start.sh config/server.properties
对于这种启动之后就可以忽略的服务，可以在最前面加上nohup，让其在后台自己运行。
### 3.3 创建话题topic
新开一个命令行窗口，创建一个叫做testkafka的topic：
   bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic testkafka
然后可以根据地址和端口号将话题topic 展示出来：
   bin/kafka-topics.sh --list --zookeeper localhost:2181
显示如下:
    testkafka
### 3.4 启动生产者producer
再开一个producer命令行窗口，执行以下命令：
   bin/kafka-console-producer.sh --broker-list localhost:9092 --topic testkafka
然后可以之间在本窗口输入消息，每遇到换行符就认为是一条消息输入完成。
   Hello kafka !
### 3.5 启动消费者consumer
   再新开一个consumer命令行窗口，执行以下命令：
   bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic testkafka --from-beginning
显示如下：
   Hello kafka !
这时候，启动kafka服务的命令行就会有显示：
  [2016-06-07 14:19:12,683] INFO Closing socket connection to /127.0.0.1. (kafka.network.Processor)

然后，每在producer那里输入一条，consumer这里就会显示一条，然后kafka服务那里也会产生日志记录。
在以后启动时，只需要依次启动kafka server，producer，consumer就可以了。


