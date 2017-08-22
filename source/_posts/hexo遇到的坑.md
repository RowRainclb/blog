---
title: hexo遇到的坑
date: 2014-11-10 17:11:46
tags:
    - hexo
    - github
---
看见别人的博客绚丽多彩，与众不同，自己也鼓捣了一个自己的博客
github+hexo搭建
首先hexo的安装，git安装，往上一大堆，废话不多说,可以参考　http://www.cnblogs.com/highway-9/p/5985893.html，下面总结一下遇到的坑
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

坑2：
换了台机器，从github pull下来代码，安装git,node,hexo后，启动hexo s,显示启动成功
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
但是界面访问显示404  Cannot GET /
解决：
有网友说进行如下操作即可
npm install
试了之后不行，有网友说进行如下操作即可：
sudo npm install hexo-renderer-ejs --save
sudo npm install hexo-renderer-stylus --save
sudo npm install hexo-renderer-marked --save
这个时候再重新生成静态文件，命令：hexo g 启动：hexo s
试了还是不行，应该还是哪些包关联出了问题，最后还是init了新文件，
把除了node_modules文件外的文件都复制过来即可：
步骤：
1. hexo init <folder>
2. cd folder
3. npm install
4. npm install hexo-server --save
5. 把之前的除掉node_modules文件外的文件复制过来(或者把node_modules文件夹替换之前的node_modules文件夹)
6. npm server
上述操作亲测可行，后来发现不用这么复杂
步骤：
sudo npm install
sudo npm install hexo-server --save
这样也是可以的



