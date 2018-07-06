---
title: 如何查找Ubuntu系统中占用磁盘最大的那些文件夹
date: 2018-02-08 14:41:25
tags:
       - ubuntu
---
# 如何查找Ubuntu系统中占用磁盘最大的那些文件夹

我们需要用df和du两个磁盘管理命令来查看
先用df来了解磁盘大致的空间情况：
![](1.jpg)
然后用du -sh 某个folder来查看哪个文件夹占用多少空间
![](2.jpg)
然后我们可以用du /homewebown | sort -nr | more 可来定位具体是哪个文件夹占用空间过大。
![](3.jpg)


转载：http://www.178linux.com/58067
