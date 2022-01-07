# Nginx在Linux环境下的安装

第一步，安装gcc的环境。



```plain
[root@nginx home]# yum install gcc-c++
```



第二步，安装第三方依赖包。



```plain
// PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。nginx的http模块使用pcre来解析正则表达式，所以需要在linux上安装pcre库。
[root@nginx home]# yum install -y pcre pcre-devel
// zlib库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip，所以需要在linux上安装zlib库。
[root@nginx home]# yum install -y zlib zlib-devel
// OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。
[root@nginx home]# yum install -y openssl openssl-devel
```



```plain
这里使用的是yum安装，也可以下载三方库离线安装
```



第三步，解压ngnix安装包。



```shell
[root@nginx home]# tar -zxvf nginx-1.8.0.tar.gz
```



第四步，使用./configure命令创建makeFile文件,并且配置nginx。

```shell
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi
```

第五步，make&&make  install



```plain
[root@nginx nginx-1.8.0]# make
[root@nginx nginx-1.8.0]# make install
```



第六步，创建临时文件。



```plain
[root@nginx home]# mkdir /var/temp/nginx/client -p
```



第七步，启动nginx。



```plain
[root@nginx home]# cd /usr/local/nginx/sbin/
// 启动
[root@nginx sbin]# ./nginx
// 关闭
[root@nginx sbin]# ./nginx -s quit
// 重启,先关闭后启动
// 动态加载配置文件
[root@nginx sbin]# ./nginx -s reload
```

# nginx.conf配置详解

```shell
# Nginx用户及组：用户 组。window下不指定，linux也可不指定;
user nginx nginx ;

# 工作进程：数目。根据硬件调整，通常等于CPU数量或者2倍于CPU。
worker_processes  1;

# 错误日志：存放路径。
error_log  logs/error.log;
error_log  logs/error.log  notice;
error_log  logs/error.log  info;

# pid（进程标识符）：存放路径。
pid        logs/nginx.pid;


events {
    # 每个工作进程的最大连接数量。根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cpu跑到100%就行。每个进程允许的最多连接数，理论上每台nginx服务器的最大连接数为。worker_processes*worker_connections
    worker_connections  1024;
}


http {
    # 设定mime类型,类型由mime.type文件定义
    include      mime.types;
    default_type  application/octet-stream;

    # 日志格式设置。
    # $remote_addr与$http_x_forwarded_for用以记录客户端的ip地址；
    # $remote_user：用来记录客户端用户名称；
    # $time_local： 用来记录访问时间与时区
    # $request： 用来记录请求的url与http协议
    # $status： 用来记录请求状态；成功是200，
    # $body_bytes_sent ：记录发送给客户端文件主体内容大小；
    # $http_referer：用来记录从那个页面链接访问过来的；
    # $http_user_agent：记录客户浏览器的相关信息；
    # 通常web服务器放在反向代理的后面，这样就不能获取到客户的IP地址了，通过$remote_add拿到的IP地址是反向代理服务器的iP地址。反向代理服务器在转发请求的http头信息中，可以增加x_forwarded_for信息，用以记录原有客户端的IP地址和原来客户端的请求的服务器地址。
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    # 用了log_format指令设置了日志格式之后，需要用access_log指令指定日志文件的存放路径；
    access_log  logs/access.log  main;

    # sendfile指令指定 nginx 是否调用sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。
    sendfile        on;
    # 此选项允许或禁止使用socke的TCP_CORK的选项，此选项仅在使用sendfile的时候使用
    tcp_nopush    on;

    # keepalive超时时间。;
    keepalive_timeout  65;

    # gzip  on;

    # nginx的upstream目前支持4种方式的分配
    # 1、轮询（默认）
    #        每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
    #    weight：指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。  
    #  例如：
     upstream bakend {
        server 192.168.0.14 weight=10;
        server 192.168.0.15 weight=10;
     }
    # 2、ip_hash
    # 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
    # 例如：
    upstream bakend {
        ip_hash;
        server 192.168.0.14:88;
        server 192.168.0.15:80;
    }
    # 3、fair（第三方）
    # 按后端服务器的响应时间来分配请求，响应时间短的优先分配。
    upstream backend {
        server server1;
        server server2;
        fair;
    }
    # 4、url_hash（第三方）
    # 按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
    # 例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
    upstream backend {
        server squid1:3128;
        server squid2:3128;
        hash $request_uri;
        hash_method crc32;
    }
    # tips:
    upstream bakend{#定义负载均衡设备的Ip及设备状态}{
        ip_hash;
        server 127.0.0.1:9090 down;
        server 127.0.0.1:8080 weight=2;
        server 127.0.0.1:6060;
        server 127.0.0.1:7070 backup;
    }
    # 在需要使用负载均衡的server中增加
    # proxy_pass http://bakend/;
    # 每个设备的状态设置为:
    # 1.down表示单前的server暂时不参与负载
    # 2.weight为weight越大，负载的权重就越大。
    # 3.max_fails：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream模块定义的错误
    # 4.fail_timeout:max_fails次失败后，暂停的时间。
    # 5.backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。
    # nginx支持同时设置多组的负载均衡，用来给不用的server来使用。
    # client_body_in_file_only设置为On 可以讲client post过来的数据记录到文件中用来做debug
    # client_body_temp_path设置记录文件的目录 可以设置最多3层目录
    # location对URL进行匹配.可以进行重定向或者进行新的代理 负载均衡
   
    # 配置虚拟主机 
    server {
        # 配置监听端口
        listen      80;
        # 配置访问域名
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            # 设置被代理服务器的端口或套接字，以及URL
            proxy_pass http://img_relay$request_uri;
            # 将代理服务器收到的用户的信息传到真实服务器上
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            root  html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page  500 502 503 504  /50x.html;
        location = /50x.html {
            root  html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass  http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root          html;
        #    fastcgi_pass  127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # 禁止访问.htxxx文件
        location ~ /\.ht {
            deny  all;
        }
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen      8000;
    #    listen      somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root  html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen      443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root  html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

# 配置虚拟主机

为什么要配置虚拟主机



```plain
就是在一台服务器启动多个网站，有两种方式：
1、域名区分；
2、端口区分。
```



端口区分



```plain
// 修改nginx配置文件
[root@nginx nginx-1.8.0]# vim /usr/local/nginx/conf/nginx.conf
```



nginx.conf



```plain
    // 同一个域名下使用端口区分不同的服务
    server {
        listen      80;
        server_name  localhost;
        location / {
            root  html;
            index  index.html index.htm;
        }
    }
    
    server {
        listen      81;
        server_name  localhost;
        location / {
            root  html81;
            index  index.html index.htm;
        }
    }
```



域名区分



```plain
    // 两个域名，使用的是相同的端口
    server {
        listen      80;
        server_name  www.manager.com;
        location / {
            root  html-manager;
            index  index.html index.htm;
        }

    }

    server {
        listen      80;
        server_name  www.api.com;
        location / {
            root  html-api;
            index  index.html index.htm;
        }
    }
```

# 配置反向代理

```shell
upstream jzweb{
    server 172.16.0.212;
}
server {
    listen      80;
    server_name  www.jzweb.com;
       
    location / {
        proxy_pass http://jzweb;
    }
}


    upstream jzshop{
        server 172.16.0.212:8080;
        server 172.16.0.209:8080;
    }

    server {
        listen      8080;
        server_name  shop.sctobacco.com;

        location / {
            proxy_pass http://jzshop;
        }
    }
```