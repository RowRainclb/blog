---
title: springboot的jar包，没有主清单属性
date: 2017-04-28 14:23:53
tags:
   - springboot
   - maven
   - jar
---
# springboot的jar包，没有主清单属性
最近开发项目时，springboot项目开发完成打成jar包，在使用java -jar test.jar 运行时报错：
k2alpha-sample.jar中没有主清单属性

解决：
在maven文件中，加入如下代码：
```javascript
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

再次运行正常
