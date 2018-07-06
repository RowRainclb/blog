---
title: 'MySQL:reading initial communication packet问题解决方法'
date: 2018-02-08 11:38:48
tags:
   - mysql
   - navicat
---
# MySQL:reading initial communication packet问题解决方法

Lost connection to MySQL server at 'reading initial communication packet' 错误解决 

上次解决了这个问题，今天又碰到，突然失忆，又做了一番无用功后终于搞定，这次一定要记录下来，免得下次又浪费时间 

1、修改mysql配置文件 

vi /etc/my.cnf 

[mysqld]段加skip-name-resolve 

在这个之前要把mysql的远程访问权限打开，或者再加skip-grant-table（不推荐） 

2、修改hosts.allow 

vi /etc/hosts.allow 

加mysqld : ALL : ALLOW 
mysqld-max : ALL :ALLOW 
其它网友的补充：
mysql教程 'reading initial communication packet'错误解决方法
出现这种问题是服务器突然关掉出现的问题，
错误提示是:
无法链接数据库教程(mysql)服务器, 请检查服务器地址、用户名、密码.
代码: 2013
错误: lost connection to mysql server at 'reading initial communication pa(www.jb51.net)cket', system error: 0
下面我们来看看具体解解决办法
方法一：解决方法是在 my.cnf 里面的 [mysqld] 段增加一个启动参数 skip-name-resolve
方法二：如果你方法一不行，可以尝试重装mysql server这样，再把数据库导进去就ok了。
总结：

如果能不重装mysql情况能把机器搞好，那是最好不过了，不在万不得己请不要重装mysql哦
