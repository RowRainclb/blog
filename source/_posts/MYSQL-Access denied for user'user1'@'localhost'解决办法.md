---
title: MYSQL:Access denied for user'user1'@'localhost'解决办法
date: 2019-08-12 20:27:54
tags:
       - mysql
---
# 前提介绍


在执行select * from table into outfile '/home/xx.txt* character set utf8 fields terminated by '' optionally enclosed by '' lines terminated by '\n'时 出现 EROOR 1045(28000) at
line 3 in file:'xx.txt':Access denied for user 'user1'@'localhost' (using password:YES)错误

取出相应内容，效率自然大大提升。对limit的优化，不是直接使用limit，而是首先获取到offset的id，然后直接使用limit size来获取数据。

# 解决办法
这个错误比较笼统，意思肯定是没有权限做某件事
1 按照这个思想，首先必须确保用户名密码正确性
2 如果是写入和读取文件，肯定要确保文件和目录有相应的权限，可以chmod 777 xx 给文件复最高权限试试
3 如果还是不行，以root用户登录Mysql，查看用户操作文件权限，富裕用户操作文件的权限
> use mysql
> select * from user\G;
> update user set File_priv='Y' where user = 'xx';

