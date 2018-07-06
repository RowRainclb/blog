---
title: 微信小程序嵌入html，界面在苹果能出来，安卓出不来
date: 2018-06-04 09:52:12
tags:
       - 微信小程序
       - web-view
---
# 微信小程序web-view嵌入html后，安卓手机正常显示，iso却是白屏

前提:最近小程序需要画非常复杂的图谱，由于小程序画图功能有限，只能以嵌入
html界面的方式展示，结果在苹果手机上无法加载，试了android手机是可以的

后来查到是由于web-view的src中携带中文参数，iso不允许链接有中文，

解决：
转码： 
```javascript
encodeURI(url)
```
解码:
```javascript
decodeURI(url)
```
