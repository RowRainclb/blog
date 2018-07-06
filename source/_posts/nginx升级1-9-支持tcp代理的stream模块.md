---
title: 'nginx升级1.9,支持tcp代理的stream模块'
date: 2018-05-08 19:47:01
tags:
       - nginx
---
# nginx升级&支持tcp/socket转发

回忆：坑的来源
    外网服务器nginx一直用的好好的，主要用于http代理和反代理，忽然有一天，客户想要外网访问内网的kafka，这样就必须
要支持tcp转发了，好吧，开始操作

前提：
nginx的安装：
sudo apt-get install nginx


    首先客户外网服务器nginx是1.4版本的，支持tcp转发必须升级到1.9以上了
 ## 升级nginx
 
   ###  查看系统信息
    lsb_release -a
    
```javascppt
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.5 LTS
Release:	14.04
Codename:	trusty
```
 ### 下载最nginx 版为1.9.12

wget http://nginx.org/download/nginx-1.9.12.tar.gz

### 解压

tar zxvf nginx-1.9.12.tar.gz

cd nginx-1.9.12 

### 查看原来的nginx信息

nginx -V

nginx version: nginx/1.4

built by gcc 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04)

built with OpenSSL 1.0.1f 6 Jan 2014

TLS SNI support enabled

configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module --with-http_auth_request_module --with-mail --with-mail_ssl_module --with-file-aio --with-http_spdy_module --with-cc-opt='-g -O2 -fstack-protector --param=ssp-buffer-size=4 -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,--as-needed' --with-ipv6

 

### .执行configure命令,后面跟上原来nginx的配置

./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module --with-http_auth_request_module --with-mail --with-mail_ssl_module --with-file-aio --with-http_spdy_module --with-cc-opt='-g -O2 -fstack-protector --param=ssp-buffer-size=4 -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,--as-needed' --with-ipv6

 

错误1

./configure: error: invalid option "--with-http_spdy_module"

根据官方文档说明需要修改为

--with-file-aio --with-http_v2_module

参考： (http://www.klfy.net/blog/636.shtml)

 

错误2

./configure: error: the HTTP rewrite module requires the PCRE library.

rewrite需要pcre支持,

apt-get install libpcre3 libpcre3-dev

 

#重新生成makefile

./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module --with-http_auth_request_module --with-mail --with-mail_ssl_module --with-file-aio --with-http_v2_module --with-cc-opt='-g -O2 -fstack-protector --param=ssp-buffer-size=4 -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,--as-needed' --with-ipv6 

 

### 编译

make

会在objs/目录下 编译生成nginx

objs/nginx -v

nginx version: nginx/1.9.12

 

 

### 备份原来的nginx到nginx1.4

mv /usr/sbin/nginx /usr/sbin/nginx1.4 

#复制新的nginx到/usr/sbin/nginx

cp objs/nginx /usr/sbin/nginx

 

### 升级命令

make upgrade 

/usr/sbin/nginx -t

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok

nginx: configuration file /etc/nginx/nginx.conf test is successful

kill -USR2 `cat /var/run/nginx.pid`

sleep 1

test -f /var/run/nginx.pid.oldbin

kill -QUIT `cat /var/run/nginx.pid.oldbin`

 

### 查看版本

nginx -v

nginx version: nginx/1.9.12

 

### 检查配置信息

/usr/sbin/nginx -t

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok

nginx: configuration file /etc/nginx/nginx.conf test is successful

 

###重启nginx

 nginx -s reload
 
 
 
 ## Nginx支持socket转发
 有个接口是通过socket通信，对端服务器访问存在IP限制，只好通过跳板机，因为它具备访问对端服务器的权限。nginx1.9开始支持tcp层的转发，通过stream实现的，而socket也是基于tcp通信。
 
 1.安装nginx，stream模块默认不安装的，需要手动添加参数：–with-stream
 2.nginx.conf 配置
 nginx.conf
 ```javascript
 user nginx;
worker_processes 1;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;
events {
worker_connections 1024;
}
http {
.................
}

# tcp层转发的配置文件夹

include /etc/nginx/tcp.d/*.conf;
 ```
 请注意，stream配置不能放到http内，即不能放到/etc/nginx/conf.d/，因为stream是通过tcp层转发，而不是http转发。
 3.在tcp.d下新建个test.conf文件，内容如下：
 ```javascript
 stream {
    # 添加socket转发的代理
    upstream bss_num_socket {
        hash $remote_addr consistent;
        # 转发的目的地址和端口
        server 130.51.11.33:19001 weight=5 max_fails=3 fail_timeout=30s;
    }

    # 提供转发的服务，即访问localhost:30001，会跳转至代理bss_num_socket指定的转发地址
    server {
       listen 30001;
       proxy_connect_timeout 1s;
       proxy_timeout 3s;
       proxy_pass bss_num_socket;
    }
}
 ```
 
 4.重启nginx
 
 
 问题来了，重启报错：
 1.nginx: [emerg] unknown directive "stream " 
 nginx没有安装stream模块，configure时添加–with-stream
 
 解决：
 重新执行上方的./configure编译，后面记得加--with-stream
 make
 make install
 
 ok!升级完成，tcp转发完成
 
 你很优秀，使劲夸我吧！～～～
 
 
 
   
    
