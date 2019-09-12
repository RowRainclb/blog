---
title: mysql慢查询优化-分页慢查询篇
date: 2019-08-12 21:37:52
tags:
       - mysql
       - 优化
---
# 前提介绍

**为何分页查询在测试环境没事，在生产上几千万的数据就出现了问题**
在平时开发时，由于数据量没有那么大，所以测试有时候会不到位，比如用到的分页查询，使用不规范时，数据量越大，查询越慢，而且有
长时间进程不结束，会导致内存不足等风险


传统分页查询：SELECT c1,c2,cn… FROM table LIMIT n,m

MySQL的limit工作原理就是先读取前面n条记录，然后抛弃前n条，读后面m条想要的，所以n越大，偏移量越大，性能就越差。
因为要取出所有字段内容，这种需要跨越大量数据块并取出



推荐分页查询方法
通过直接根据索引字段定位后，才取出相应内容，效率自然大大提升。对limit的优化，不是直接使用limit，而是首先获取到offset的id，然后直接使用limit size来获取数据。

## 1、尽量给出查询的大致范围

SELECT c1,c2,cn... FROM table WHERE id>=20000 LIMIT 10;
## 2、子查询法

SELECT c1,c2,cn... FROM table WHERE id>=
(
SELECT id FROM table LIMIT 20000,1
)
LIMIT 10;
## 3、子查询2

SELECT * FROM product a JOIN (select id from product limit 866613, 20) b ON a.ID = b.id
## 3、高性能MySQL一书中提到的只读索引方法

优化前SQL:

SELECT c1,c2,cn... FROM member ORDER BY last_active LIMIT 50,5
优化后SQL:

SELECT c1, c2, cn .. .
FROM member
INNER JOIN (SELECT member_id FROM member ORDER BY last_active LIMIT 50, 5)
USING (member_id)
分别在于，优化前的SQL需要更多I/O浪费，因为先读索引，再读数据，然后抛弃无需的行。而优化后的SQL(子查询那条)只读索引(Cover index)就可以了，然后通过member_id读取需要的列。


