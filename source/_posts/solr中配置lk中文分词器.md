---
title: solr中配置lk中文分词器
date: 2014-11-22 13:50:20
tags:
    - solr
    - lk
    - 分词
---
配置IK中文分词器
以collection1为例。
1.	将IKAnalyzer2012FF_u1.jar拷贝到tomcat\webapps\solr\WEB-INF\lib目录下。
2.	将IKAnalyzer.cfg.xml和stopword.dic拷贝到tomcat\webapps\solr\WEB-INF\classes目录下。
3.	修改solr/home下的collection1/conf/scheme.xml文件。在该文件中找到<schema></schema>部分，在中间插入：
``` javascript
<fieldType name="text_ik" class="solr.TextField">
      	<analyzer type="index" isMaxWordLength="false" class="org.wltea.analyzer.lucene.IKAnalyzer"/>
	<analyzer type="query" isMaxWordLength="true" class="org.wltea.analyzer.lucene.IKAnalyzer"/>
</fieldType>
```
4.	将scheme.xml中的<field>名称为“title”的type类型改为”text_ik“。
5.	重启solr服务器，在地址栏中执行http://localhost:8080/solr/#/collection1/analysis <http://localhost:8080/solr/>，随便输入一句话，进行测试：
 ![](solr中配置lk中文分词器/lk.png)
出现以上界面，说明分词器配置成功。



