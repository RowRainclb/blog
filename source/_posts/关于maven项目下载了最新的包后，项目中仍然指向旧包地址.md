---
title: 关于maven项目下载了最新的包后，项目中仍然指向旧包地址
date: 2017-12-06 14:44:35
tags:
       - maven
       - intellij
---
# 解决使用maven项目开发时，mvn install后下载了新的依赖包，程序中却还是指向旧的依赖包的问题
  
   描述：
   最近同事遇到一个无法获取最新maven依赖的问题，这个问题我之前也遇到过，做下记录
   maven项目 a 中引入了项目b的依赖
   项目b增加了新的方法，部署到了maven仓库
   项目b执行，mvn install后下载下来了最新的项目b
   但是项目a中还是无法调用项目b的新方法
   
   
   解决：
   由于我们使用的开发工具是intellij IDEA，所以在执行mvn install后，依赖下载到本地仓库后，还需要重新导入依赖
   2中方式
   方式一
   ![](area.jpg)
  方式二
  ![](menu.jpg)