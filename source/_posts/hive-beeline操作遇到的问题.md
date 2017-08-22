---
title: hive beeline操作遇到的问题
date: 2016-06-22 16:27:56
tags:
---
1 Org.apache.hadoop.hive.service.ThriftHive
![](hive-beeline操作遇到的问题/hive1.jpg)
1 找不到org.apache.hive.jdbc.HiveDriver  （升级到hive-jdbc-0.14.0.jar） 
2 org.apache.hive.service.cli.thrift.TCLIService（升级到hive-service-0.14.0.jar） 
3 org.apache.thrift.TServiceClient（升级到hive-exec-0.14.0.jar） 
以上问题基本由于版本导致（升级后如图）：
![](hive-beeline操作遇到的问题/hive2.jpg)
2 beeline -u jdbc:hive://localhost:10000 -n hive 
报错：no known driver to handle "jdbc:hive://localhost:10000" 
![](hive-beeline操作遇到的问题/hive3.jpg)
解决办法1： 
beeline -u jdbc:hive2://localhost:10000 -n hive 
解决办法2： 
beeline -u jdbc:hive2://localhost:10000 -d org.apache.hive.jdbc.HiveDriver 
 -n hive 
