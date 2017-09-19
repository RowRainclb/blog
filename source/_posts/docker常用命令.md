---
title: docker常用命令
date: 2017-04-11 14:10:04
tags:
    - docker
---
# docker 的各种命令和参数
  docker images --查看本地镜像
  docker ps  -- 查看正在运行的容器
  docker ps -a  --查看所有的容器

1 rm --删除容器，注意，不可以删除一个运行中的容器，必须先用docker stop或docker kill使其停止。
  当然可以强制删除，必须加-f参数
  如果要一次性删除所有容器，可使用 docker rm -f `docker ps -a -q`，其中，-q指的是只列出容器的ID
2 rmi --删除镜像
3 run --让创建的容器立刻进入运行状态，该命令等同于docker create创建容器后再使用docker start启动容器
```javascript
Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]    
  
  -d, --detach=false         指定容器运行于前台还是后台，默认为false     
  -i, --interactive=false   打开STDIN，用于控制台交互    
  -t, --tty=false            分配tty设备，该可以支持终端登录，默认为false    
  -u, --user=""              指定容器的用户    
  -a, --attach=[]            登录容器（必须是以docker run -d启动的容器）  
  -w, --workdir=""           指定容器的工作目录   
  -c, --cpu-shares=0        设置容器CPU权重，在CPU共享场景使用    
  -e, --env=[]               指定环境变量，容器中可以使用该环境变量    
  -m, --memory=""            指定容器的内存上限    
  -P, --publish-all=false    指定容器暴露的端口    
  -p, --publish=[]           指定容器暴露的端口   
  -h, --hostname=""          指定容器的主机名    
  -v, --volume=[]            给容器挂载存储卷，挂载到容器的某个目录    
  --volumes-from=[]          给容器挂载其他容器上的卷，挂载到容器的某个目录  
  --cap-add=[]               添加权限，权限清单详见：http://linux.die.net/man/7/capabilities    
  --cap-drop=[]              删除权限，权限清单详见：http://linux.die.net/man/7/capabilities    
  --cidfile=""               运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法    
  --cpuset=""                设置容器可以使用哪些CPU，此参数可以用来容器独占CPU    
  --device=[]                添加主机设备给容器，相当于设备直通    
  --dns=[]                   指定容器的dns服务器    
  --dns-search=[]            指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件    
  --entrypoint=""            覆盖image的入口点    
  --env-file=[]              指定环境变量文件，文件格式为每行一个环境变量    
  --expose=[]                指定容器暴露的端口，即修改镜像的暴露端口    
  --link=[]                  指定容器间的关联，使用其他容器的IP、env等信息    
  --lxc-conf=[]              指定容器的配置文件，只有在指定--exec-driver=lxc时使用    
  --name=""                  指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字    
  --net="bridge"             容器网络设置:  
                                bridge 使用docker daemon指定的网桥       
                                host    //容器使用主机的网络    
                                container:NAME_or_ID  >//使用其他容器的网路，共享IP和PORT等网络资源    
                                none 容器使用自己的网络（类似--net=bridge），但是不进行配置   
  --privileged=false         指定容器是否为特权容器，特权容器拥有所有的capabilities    
  --restart="no"             指定容器停止后的重启策略:  
                                no：容器退出时不重启    
                                on-failure：容器故障退出（返回值非零）时重启   
                                always：容器退出时总是重启    
  --rm=false                 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)    
  --sig-proxy=true           设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理 

```
4 save --将镜像打包
5 search --从Docker Hub中搜索镜像
6 start --启动容器
7 stats --动态显示容器的资源消耗情况，包括：CPU、内存、网络I/O
8 stop --停止一个运行的容器
9 tag --对镜像进行重命名
10 top --查看容器中正在运行的进程
11 unpause --恢复容器内暂停的进程
12 version --查看docker的版本
13 wait  --捕捉容器停止时的退出码


案例1、运行一个简单的容器，其中需要包含控制台管理
```javascript
[root@CentOS7.2 ~]#docker run -i -t centos6.8
```
这个容器一执行就会进入到默认的线程”/bin/bash”，直接进入控制台操作。当退出控制后后，容器会被终止。

案例2、运行一个在后台执行的容器，同时，还能用控制台管理
```javascript
[root@CentOS7.2 ~]#docker run -i -t -d centos6.8 
```

案例3、运行一个带命令在后台不断执行的容器，不直接展示容器内部信息
```javascript
[root@CentOS7.2 ~]#docker run -d centos6.8  ping www.docker.com  
```
这个容器将永久在后台执行，因为ping这个线程不会停止。除非你停止了ping的线程。

案例4、运行一个在后台不断执行的容器，同时带有命令，程序被终止后还能重启继续跑，还能用控制台管理
```javascript
    [root@CentOS7.2 ~]#docker run -d --restart=always centos6.8  ping www.docker.com  
```
这个容器将永久在后台执行，因为ping这个线程不会停止。如果你把ping这个线程终止了，那么容器会重启继续执行ping功能

案例5、我们需要为容器指定一个名称
```javascript
[root@CentOS7.2 ~]#docker run -d --name=server-db centos6.8-mysql /usr/bin/mysql_safe -d 
```
这时候我们这个容器的名称为server-db，同时激活了数据库mysql的后台线程，让它不断的跑，这时候我们的容器也不会被关闭。

案例6、我们需要让server-http容器连接server-db容器
```javascript
[root@CentOS7.2 ~]#docker run -d --name=server-http --link=server-db  centos6.8-httpd /usr/bin/httpd --DFOREGROUND	
```
这时候，我们执行了apache的服务器让它不断的在后台执行，同时，在php里配置mysql的服务器名称为”server-db”，直接用server-db命名就可以了。不需要输入ip地址之类的。我们的server-http指定连接了server-db。server-db在server-http里会被当做一个DNS解析来获取相应的连接ip。

案例7、我们要将server-db,server-http的端口暴露出去，让大家访问
```javascript
    [root@CentOS7.2 ~]#docker run -d --name=server-db -p 3306:3306 centos6.8-mysql /usr/bin/mysql_safe –d  
```
这时候我们指定了服务器宿主机的3306端口映射到容器的3306端口，暴露出去。


案例8、我们要将宿主机的数据库目录/server/mysql-data挂载到server-db上
```javascript
[root@CentOS7.2 ~]#docker run -d --name=server-db -p 3306:3306 -v /server/mysql-data:/mysql-data centos6.8-mysql /usr/bin/mysql_safe –d  
```
这时候，你会发现，在server-db根目录下你会发现有一个新的文件夹mysql-data，同时里面的文件内容和宿主机下/server/mysql-data一样。

案例9、我们希望一个容器在它的进程结束后，立马自动删除。
```javascript
[root@CentOS7.2 ~]#docker run -it --rm  centos6.8 
```
这时候我们进入了容器的控制台，当我们在容器内部exit退出控制台的时候，容器将被终止，同时自动删除。 

以上内容参考：http://blog.csdn.net/kunloz520/article/details/53839237

补充：
案例10： 现在您已经有了一个被修改过的 Container，记下这个 Container 的 ID，现在您可以使用 docker commit 命令将此 Container 的副本提交到一个镜像里
```javascript
docker commit -m "Added connect and serve-static" -a "backslash112" 0b2616b0e5a8 backslash112/node:v1
```


案例11： 将镜像推送到 Docker Hub 
使用 docker push <image> 命令可以将一个镜像推送到 Docker Hub 服务器的您的帐号下（类似 Github）。
```javascript
docker push backslash112/node:v1
```
此时您可以拿来和别人共享或者设置为私有仓库。

案例12： 利用 Docker 在另一台机器上快速部署
通过 Github 将 nodejs 项目同步到服务器，然后在服务器中执行以下命令 
```javascript
docker run -it --name my-server -v $(pwd):/dev_carl -w /dev_carl -p 8080:8080 backslash112/node:v1 node server.js
```



推荐阅读：
[Docker：带给现代开发人员的福利](https://www.ibm.com/developerworks/cn/web/wa-docker-polyglot-programmers/)
 













