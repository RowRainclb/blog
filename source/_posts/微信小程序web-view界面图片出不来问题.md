---
title: 微信小程序web-view界面图片出不来问题
date: 2018-07-05 15:14:15
tags:
       - 微信小程序
       - web-view
       - 问题
---
# 微信小程序使用web-view嵌入界面, 界面中图片出不来解决方案

## 坑一：空白界面
```javascript

<web-view src="{{url}}">
</web-view>
```

如上代码所示，url 默认为'' ,界面进来是空白界面
原因：web-view进来按照url默认值加载后不会再刷新界面，即使url赋值后也不会出来

添加判断语句，修改如下：
```javascript
<block wx:if="{{url != ''}}">
<web-view src="{{url}}">
</web-view>
</block>
```

## 坑二：图片地址当参数传如界面，图片出不来
js代码如下：

```javascript
page({
 data: {
  url: ''
 }
 onLoad: function() {
   // 获取到的photoUrl
   var photoUrl = 'https://xxxx';
   // 作为参数进行转码
  var photoUrlParam = encodeURIComponent(photoUrl);
   //给url动态赋值
   this.setData({
     url: "https://www.xxxxxx?photoUrl="+ photoUrlParam
   })
 }
})
```

wxml代码如下：
```javascript
<block wx:if="{{url != ''}}">
<web-view src="{{url}}">
</web-view>
</block>

```

问题：
如上，图片作为参数传界面后，浏览器能打开，小程序中图片不显示

解决：
动态传参的方式

代码如下：
```javascript
page({
 data: {
  photoUrlParam: ''
 }
 onLoad: function() {
   // 获取到的photoUrl
   var photoUrl = 'https://xxxx';
   // 作为参数进行转码
  var photoUrlParam = encodeURIComponent(photoUrl);
   //给url动态赋值
   this.setData({
     photoUrlParam:  photoUrlParam
   })
 }
})
```

wxml代码如下：
```javascript
<web-view src="https://www.xxxxxx?photoUrl={{photoUrlParam}}">
</web-view>

```