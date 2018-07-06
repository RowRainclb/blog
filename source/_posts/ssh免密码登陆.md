---
title: ssh免密码登陆
date: 2017-12-14 19:34:24
tags:
       - ssh
---
# ssh免密码登陆方式

## 本机免密码登陆本机测试：
ssh-keygen -t rsa
一路回车
在~/.ssh 生成2个文件，将公匙内容添加到authorized_keys
cp  ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
ssh localhost
如果不行则：ssh-add
成功！

##  添加到服务器：
将本地公匙文件内容添加到服务器的
cat ~/.ssh/id_rsa.pub | ssh tester@11.11.11.11 "mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys; chmod 600 ~/.ssh/authorized_keys" 

