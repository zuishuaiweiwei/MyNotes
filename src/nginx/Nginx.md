# Nginx

​	直接使用yun进行安装

​	nginx -tc nginx.conf  检查配置文件是否有错

​	nginx -s reload -c nginx.conf 从新加载nginx

## 一、安装目录

​	安装完成之后 rpm -ql 列出所有文件安装的位置

```shell
# 配置文件，用于Nginx的日志轮转，logrotate服务的日志切割，日志生成的模式 类似log4j每天生成一个文件还是什么
/etc/logrotate.d/nginx
#目录 配置文件 Nginx的主配置文件
/etc/nginx
/etc/nginx/conf.d
/etc/nginx/conf.d/default.conf
/etc/nginx/nginx.conf
# 配置文件 编码转换映射的转化文件
/etc/nginx/koi-utf
/etc/nginx/koi-win
/etc/nginx/win-utf
# 配置文件 设置http协议contentType与扩展名的对应关系
/etc/nginx/mime.types
/etc/nginx/modules
# 配置文件 cgi配置相关 fastcig配置
/etc/nginx/fastcgi_params
/etc/nginx/scgi_params
/etc/nginx/uwsgi_params
# 守护进程
/etc/rc.d/init.d/nginx
/etc/rc.d/init.d/nginx-debug
/etc/sysconfig/nginx
/etc/sysconfig/nginx-debug
# 目录 Nginx模块目录
/usr/lib64/nginx
/usr/lib64/nginx/modules
# 命令 Nginx服务启动管理的终端命令
/usr/sbin/nginx
/usr/sbin/nginx-debug
#文件 目录 帮助手册
/usr/share/doc/nginx-1.16.1
/usr/share/doc/nginx-1.16.1/COPYRIGHT
/usr/share/man/man8/nginx.8.gz
/usr/share/nginx
/usr/share/nginx/html
/usr/share/nginx/html/50x.html
/usr/share/nginx/html/index.html
# 缓存服务目录
/var/cache/nginx
# Nginx日志目录
/var/log/nginx

```

## 二、编译参数

```shell
# 查看编译参数
nginx -V
#
[root@weiwei sbin]# nginx -V
nginx version: nginx/1.16.1
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-23) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: 
## 安装基础路径
--prefix=/etc/nginx 
--sbin-path=/usr/sbin/nginx 
--modules-path=/usr/lib64/nginx/modules 
--conf-path=/etc/nginx/nginx.conf 
--error-log-path=/var/log/nginx/error.log 
--http-log-path=/var/log/nginx/access.log 
--pid-path=/var/run/nginx.pid 
--lock-path=/var/run/nginx.lock 
# 执行对应模块时 Nginx保留的临时文件目录
--http-client-body-temp-path=/var/cache/nginx/client_temp 
--http-proxy-temp-path=/var/cache/nginx/proxy_temp 
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp 
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp 
--http-scgi-temp-path=/var/cache/nginx/scgi_temp 
# 设置用户和用户组
--user=nginx 
--group=nginx 
# 默认开启的模块
--with-compat 
--with-file-aio 
--with-threads 
--with-http_addition_module 
--with-http_auth_request_module 
--with-http_dav_module 
--with-http_flv_module 
--with-http_gunzip_module 
--with-http_gzip_static_module 
--with-http_mp4_module 
# 目录中随机选择一个主页
--with-http_random_index_module 

--with-http_realip_module 
--with-http_secure_link_module 
--with-http_slice_module 
--with-http_ssl_module 
# 查看nginx的状态
--with-http_stub_status_module 
# http内容替换
--with-http_sub_module 
--with-http_v2_module 
--with-mail 
--with-mail_ssl_module 
--with-stream 
--with-stream_realip_module 
--with-stream_ssl_module 
--with-stream_ssl_preread_module 
# 设置额外的参数会添加到CFLAGS变量
--with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic -fPIC' 
# 设置附加的参数，链接系统库
--with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'

```

## 三、主配置文件详解

​	/etc/nginx 是nginx的主目录 nginx是主配置文件

```shell
# vi /etc/nginx/nginx.conf

#设置nginx服务的系统使用用户
user  nginx;
# 工作进程数
worker_processes  1;
# nginx的错误日志 warn 日志级别
error_log  /var/log/nginx/error.log warn;
#  nginx启动时的pid
pid        /var/run/nginx.pid;

# 事件模块
events {
	#每个进程最大连接数
    worker_connections  1024;
}


http {
	# 子配置文件
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

	# 显示日志的格式 main是id access-log后面指定id
	# remote_addr	服务端地址
	# remote_user	服务端认证的用户，没开启则没有
	# request	请求行
	# status	response的返回状态
	# body_bytes_sent	响应给客户的body的大小
	# http_referer	请求来源
	# http_user_agent	客户端用什么访问
	# http_x_forwarded_for	
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
	
    sendfile        on;
    #tcp_nopush     on;
	# 超时时间
    keepalive_timeout  65;

    #gzip  on;
	# 子配置文件 刚安装完有一个默认的default默认配置
    include /etc/nginx/conf.d/*.conf;
}

```

​	

```shell
# vi default.conf 
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
    	#访问根目录时在root目录下找index的指定的页面
  
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;

```

## 四、日志文件

```shell
# 查看错误日志，在主配置文件里指定路径

tail -f /var/log/nginx/error.log
#
[root@weiwei conf.d]# tail -f /var/log/nginx/error.log 
2019/08/19 11:47:21 [error] 25132#25132: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.214.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.214.30"

# 查看访问日志，在主配置文件里指定路径
tail -f /var/log/nginx/access.log
# 日志格式在主配置文件里指定格式
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
#
[root@weiwei nginx]# tail -f -n 5 /var/log/nginx/access.log
192.168.214.1 - - [19/Aug/2019:12:10:32 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
192.168.214.1 - - [19/Aug/2019:12:10:32 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
192.168.214.1 - - [19/Aug/2019:12:15:22 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
192.168.214.1 - - [19/Aug/2019:12:15:22 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
192.168.214.1 - - [19/Aug/2019:12:15:22 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"

```

​	log_format 指定日志格式，后面可以写nginx自带的一些参数，也可以自己定义参数，基本规则

```shell
# request里的参数 如 http_user_agent 字母全小写，下划线连接
$http_HEADER
# response里的参数
$send_http_HEADER
```

## 五、模块

​	在查看编译参数时后面有很多with的东西，那些都是默认加载的模块。

### 1、http_stub_status_module

​	这个模块可以显示nginx的一些状态信息

```shell
#
vim /etc/nginx/conf.d/default.conf 
#在server里加入新location配置新加的模块

    location /status {
        stub_status;
    }
#修改完之后保存
#检查配置文件是否有错
nginx -tc nginx.conf 
#从新加载nginx
nginx -s reload -c nginx.conf 
#访问ip地址加配置的路径
Active connections: 1 
server accepts handled requests
 4 4 11 
Reading: 0 Writing: 1 Waiting: 0 
```

### 2、http_random_index_module 

​	目录中随机选择一个页面作为主页

```shell
    location / {
    	#默认：访问根目录时在root目录下找index的指定的页面
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
 #修改之后   
        location / {
        root   /usr/share/nginx/html;
        #开启模块，在root目录下随机一个页面作为主页
        random_index on;
        #index  index.html index.htm;
    }
```

### 3、http_sub_module 

​	html页面内容替换

```shell
#sub_filter 默认没开启
#可以写在http、server、location里面，写在http里是总的，里面的server都会使用
#sring 需要替换的内容 replacement 替换后的内容
sub_filter string replacement;
#可以写在http、server、location里面，写在http里是总的，里面的server都会使用
#http协议里的last_modified 文件最后的修改时间，如果修改时间没变 从缓存里取出页面数据
sub_filter_last_modified on|off

##可以写在http、server、location里面，写在http里是总的，里面的server都会使用
#默认开启 
#开启后只匹配html里第一个匹配到的，否则找到全部可以匹配到的
sub_filter_once on|off

    location / {
        root   /opt/html;
        index  index.html index.htm;
        sub_filter 'Welcome' 'wwww';
        sub_filter_once off;
    }

```

### 4、http_access_module

​	基于IP的访问控制模块

​	局限性：如果用户不是直接访问服务器地址，而是通过vpn或者其他代理来间接访问，就不能控制到用户的IP。

​	修改局限性的方式：

​	（1）x_fowarded_for

​	可以用到x_fowarded_for，x_forwarded_for会记录没一个经过的IP,形成IP链，但还是不够保险，想）forwarded_for只是一个头信息，可以被修改，甚至不存在

​	（2）geo模块

​	（3）自定义变量传递

```shell
#配置在http，server，location，limit_except
#CIDR 按照网段的方式运行访问
#liunx 中使用
#允许所有
allow address | CIDR |unix |all;
#相反配置
deny address | CIDR |unix |all;
	#~ 表示模式匹配
    location ~ ^/index.html {
        root   /opt/html;
        index  index.html index.htm;
        deny ip;
        allow ip;
    }
```

 