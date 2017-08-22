---
title: ubuntu系统用户名密码忘记解决办法
date: 2017-06-13 21:18:24
tags:
    - ubuntu
---
1、开机点击ESC，进去GUN GRUB界面
2、选择有recovery mode的选项，按e进入命令行
3、找到有recovery nomodeset的行，删除recovery nomodeset，并在本行末尾加上quiet splash rw init=/bin/bash，按F10；
4、在命令行输入passwd +用户名，修改密码，若修改成功，则会返回password updated successfully
16.04亲测有效

