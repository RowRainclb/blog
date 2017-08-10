---
title: hexo遇到的坑
date: 2017-08-10 17:11:46
tags:
    - hexo
    - github
---
看见别人的博客绚丽多彩，与众不同，自己也鼓捣了一个自己的博客
github+hexo搭建
首先hexo的安装，git安装，往上一大堆，废话不多说，下面总结一下遇到的坑
坑１:
更改主题后发布到github后，查看效果只有框架，一片白，无css效果，f12查看有报错信息，找不到js,css文件
解决：
进入next主题的source目录，将vendors文件的文件名改成任意其他名字，如：VEN。
　　在配next主题的配置文件_config.yml中,将vendors: 块中的_internal: vendors项改成前面重命名文件夹的名称,如_internal:VEN，保存。
　　输入命令：
        hexo clean 
        hexo g  
        hexo d
但是我更改后，发现还是没效果，仍然报错，后来f12查看报错的url后发现，提示找不到/blog/*.js，原来去/blog路径下面找文件了，但是github上js,css并不在blog目录下，而是属于第一级目录，
原来是因为我本地为了好看把访问地址改为了/http://localhost:4000/blog
打开_config.yml文件,修改root:/blog 为 root: /，问题解决

