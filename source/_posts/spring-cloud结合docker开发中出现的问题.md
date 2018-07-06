---
title: spring-cloud结合docker开发中出现的问题
date: 2017-12-19 21:34:22
tags:
    - spring-boot
    - docker
    - docker-compose
---
# docker commit新的镜像后，docker-compose启动后，配置未生效
### 前提介绍：
当前项目用spring-boot结合spring-cloud开发，功能分为多个模块，部署时，每个模块一个docker镜像，其中有一个模块叫device,
由于项目迭代需要，数据的更新频率较快，需要启动2套device模块，代码完全相同，分别访问不同的数据库, 分别为device-v1 ,device-v2
前端模块也启动2个容器，映射2个端口,ui-v1版本端口3000，ui-v2版本端口3100,ui-v1通过配置/v1访问device-v1，ui-v2通过/v2访问device-v2

说了这么多，其实就是向启动2个容器，赋予不同的配置
在尝试的过程中，一开始我在docker-compose.yml文件中加了一个服务模块ui-v2
只把ui-v1的配置项HOST_OF_API由v1改为了v2
如下所示：
``` javascript
   ui-v1:
        container_name: k2alpha-ui
        image: ui:dev-0.1.0
        restart: always
        volumes:
            - /etc/localtime:/etc/localtime:ro
            - /etc/timezone:/etc/timezone:ro
        ports:
            - 3100:3100
        environment:
            PORT: 3100
            HOST_OF_API: http://192.168.11.11:8770/v1/

    ui-v2:
        container_name: ui-v2
        image: dev.k2data.com.cn:5001/k2data/k2alpha-ui:dev-0.2.0
        restart: always
        volumes:
            - /etc/localtime:/etc/localtime:ro
            - /etc/timezone:/etc/timezone:ro
        ports:
            - 3111:3100
        environment:
            USE_CAS: 'false'
            PORT: 3100
            HOST_OF_API: http://192.168.120.8:8770/v2/
```
然后我用docker ps查看了当前启动的容器
再docker commit id ui-v2 保存了一份新的镜像
再docker-compose up -d ui-v2启动新的镜像

启动后，访问localhost:3111后发现，ui-v2的配置还是/v1
纳闷了很久，询问了公司其他同事后仍然无果
最后，下楼抽了一个烟，缓了缓，忽然一个头绪袭来，赶紧扔了烟上楼试了试，果然成了！

其实代码完全相同，为什么还要再保存一份镜像呢，直接一个镜像启动2个容器即可，container_name不同就可以了嘛
哎，终于localhost:3111可以取到配置/v2了

至于为什么docker commit新镜像后不可以，还是不知起因，解决了问题，也遗留了问题，希望有天可以碰到个大神给解释一下