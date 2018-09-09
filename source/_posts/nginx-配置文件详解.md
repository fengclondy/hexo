---
layout: post
title: nginx配置文件超详解
date: 2018-02-05 02:09:39
tags: nginx
categories: nginx
---

#### nginx的基本配置nginx.conf

##### 基本组成
配置文件主要由四部分组成：
main(全区设置)，server(主机配置)，upstream(负载均衡服务器设置)，和location(URL匹配特定位置设置)

##### 1、全局配置
```powershell
#nginx的worker进程运行用户以及用户组
user nobody;    
#就是用浏览器打开时的默认用户，如果文件不可以读取（默认访问是其他用户读取）,将其改成一致即可
#启动进程,通常设置成和cpu的数量相等
worker_processes  1;
#worker_processes auto;
#以下参数指定了哪个cpu分配给哪个进程，一般来说不用特殊指定。如果一定要设的话，用0和1指定分配方式.
#这样设就是给1-4个进程分配单独的核来运行，出现第5个进程是就是随机分配了。eg:
#worker_processes 4     #4核CPU 
#worker_cpu_affinity 0001 0010 0100 1000

#定义全局错误日志定义类型，[debug|info|notice|warn|crit]
#error_log  logs/error.log;
#error_log  logs/notice.log  notice;
#error_log  logs/info.log  info;
#指定进程ID存储文件位置
#pid        logs/nginx.pid;
#一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n的值保持一致。
#vim /etc/security/limits.conf
#  *                soft    nproc          65535
#  *                hard    nproc          65535
#  *                soft    nofile         65535
#  *                hard    nofile         65535
```

<!-- more -->

##### 2、事件配置

```conf
#工作模式及连接数上限
events {
    #use [ kqueue | rtsig | epoll | /dev/poll | select | poll ] 是多路复用IO(I/O Multiplexing)中的方式
    #epoll 仅用于linux2.6以上内核,可以大大提高nginx的性能
    use   epoll; 

    #单个后台worker process进程的最大并发链接数  
    worker_connections  1024;

    # 并发总数是 worker_processes 和 worker_connections 的乘积
    # 即 max_clients = worker_processes * worker_connections
    # 在设置了反向代理的情况下，max_clients = worker_processes * worker_connections / 4  为什么
    # 为什么上面反向代理要除以4，应该说是一个经验值
    # 根据以上条件，正常情况下的Nginx Server可以应付的最大连接数为：4 * 8000 = 32000
    # worker_connections 值的设置跟物理内存大小有关
    # 因为并发受IO约束，max_clients的值须小于系统可以打开的最大文件数
    # 而系统可以打开的最大文件数和内存大小成正比，一般1GB内存的机器上可以打开的文件数大约是10万左右
    # 我们来看看360M内存的VPS可以打开的文件句柄数是多少：
    # $ cat /proc/sys/fs/file-max
    # 输出 34336
    # 32000 < 34336，即并发连接总数小于系统可以打开的文件句柄总数，这样就在操作系统可以承受的范围之内
    # 所以，worker_connections 的值需根据 worker_processes 进程数目和系统可以打开的最大文件总数进行适当地进行设置
    # 使得并发总数小于操作系统可以打开的最大文件数目
    # 其实质也就是根据主机的物理CPU和内存进行配置
    # 当然，理论上的并发总数可能会和实际有所偏差，因为主机还有其他的工作进程需要消耗系统资源。
    # ulimit -SHn 65535
}
```

##### 3、服务器设置
```conf
http {
    #文件扩展名与文件类型映射表
    include    mime.types;
    
    #默认文件类型
    default_type  application/octet-stream;
    
    #设定日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    #定义日志的格式。后面定义要输出的内容。
    #1.$remote_addr 与$http_x_forwarded_for 用以记录客户端的ip地址；
    #2.$remote_user ：用来记录客户端用户名称；
    #3.$time_local ：用来记录访问时间与时区；
    #4.$request  ：用来记录请求的url与http协议；
    #5.$status ：用来记录请求状态； 
    #6.$body_bytes_sent ：记录发送给客户端文件主体内容大小；
    #7.$http_referer ：用来记录从那个页面链接访问过来的；
    #8.$http_user_agent ：记录客户端浏览器的相关信息
    #连接日志的路径
    access_log  logs/access.log  main;
    
    #只记录更为严重的错误日志，减少IO压力
    #error_log logs/error.log crit;
    
    #关闭日志
    #access_log  off;
    
    #默认编码
    #charset utf-8;
    
    #服务器名字的hash表大小
    #server_names_hash_bucket_size 128;
    
    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，
    #对于普通应用，必须设为 on,
    #如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，
    #以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile   on;
    
    #防止网络阻塞
    #tcp_nopush    on;
    tcp_nodelay    on;  

    #连接超时时间，单位是秒
    #keepalive_timeout  0;
    keepalive_timeout  65;
    
    #客户端请求头读取超时时间
    #client_header_timeout 10;
    #设置客户端请求主体读取超时时间
    #client_body_timeout 10;
    #响应客户端超时时间
    #send_timeout 10;
    
    #gzip模块设置
    #开启gzip压缩输出
    gzip on; 
    #最小压缩文件大小
    #gzip_min_length 1k; 
    #压缩缓冲区
    #gzip_buffers 4 16k;
    #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    #gzip_http_version 1.0;
    #压缩等级 1-9 等级越高，压缩效果越好，节约宽带，但CPU消耗大
    #gzip_comp_level 2;
    #压缩类型，默认就已经包含text/html，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    #gzip_types text/plain application/x-javascript text/css application/xml;
    #前端缓存服务器缓存经过压缩的页面
    #gzip_vary on;

    #设定请求缓冲
    #客户端请求单个文件的最大字节数
    #client_max_body_size 8m;
    #指定来自客户端请求头的hearerbuffer大小
    client_header_buffer_size    128k;
    #指定客户端请求中较大的消息头的缓存最大数量和大小。
    large_client_header_buffers  4 128k;
    
    #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。
    #fastcgi_connect_timeout 300;
    #fastcgi_send_timeout 300;
    #fastcgi_read_timeout 300;
    #fastcgi_buffer_size 64k;
    #fastcgi_buffers 4 64k;
    #fastcgi_busy_buffers_size 128k;
    #fastcgi_temp_file_write_size 128k;
    
    #以下server可包含文件夹下的配置文件
　  #include conf.d/*.conf;
```

##### 4、虚拟主机基本设置
```conf
    #设定虚拟主机配置
    server {
        #侦听端口
        listen    80;
        #访问域名。可以有多个，用空格隔开
        #server_name www.jd.com jd.com;
        server_name  www.nginx.cn;
        #编码格式，若网页格式与此不同，将被自动转码
        #charset koi8-r;

        #定义服务器的默认网站根目录位置
        root html;

        #设定本虚拟主机的访问日志
        access_log  logs/nginx.access.log  main;

        #默认请求(映射，可配置成浏览器访问)
        location / {
            
            #定义映射的路径（还有一种是alias，这里不做详讨）
            root  /home/jit/education_app/resource/IA-Images;
            #定义首页索引文件的名称。以下按顺序匹配
            index index.php index.html index.htm;  
            #开启目录浏览功能
            autoindex on;
            #显示出文件的确切大小
            autoindex_exact_size off;
            #显示的文件时间为文件的服务器时间
            autoindex_localtime on; 

        }

        # 定义错误提示页面
        error_page   500 502 503 504 /50x.html;
        location = /50x.html {
        }

        #静态文件，nginx自己处理
        location ~ ^/(images|javascript|js|css|flash|media|static)/ {
            
            #过期30天，静态文件不怎么更新，过期可以设大一点，
            #如果频繁更新，则可以设置得小一点。
            expires 30d;
        }

        #PHP 脚本请求全部转发到 FastCGI处理. 使用FastCGI默认配置.
        location ~ .php$ {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include fastcgi_params;
        }

        #禁止访问 .htxxx 文件
            location ~ /.ht {
            deny all;
        }
        
        #HTTPS虚拟主机定义
        # HTTPS server
        #
        #server {
        #    listen       443 ssl;
        #    server_name  localhost;
        #    ssl_certificate      cert.pem;
        #    ssl_certificate_key  cert.key;
        #    ssl_session_cache    shared:SSL:1m;
        #    ssl_session_timeout  5m;
        #    ssl_ciphers  HIGH:!aNULL:!MD5;
        #    ssl_prefer_server_ciphers  on;
        #    location / {
        #        root   html;
        #        index  index.html index.htm;
        #    }
        #}
    }
```

##### 5、其他配置

反向代理：
```powershell
#以下配置追加在HTTP的全局变量中

#nginx跟后端服务器连接超时时间(代理连接超时)
proxy_connect_timeout      5;
#后端服务器数据回传时间(代理发送超时)
proxy_send_timeout         5;
#连接成功后，后端服务器响应时间(代理接收超时)
proxy_read_timeout         60;
#设置代理服务器（nginx）保存用户头信息的缓冲区大小
proxy_buffer_size          16k;
#proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
proxy_buffers              4 32k;
#高负荷下缓冲大小（proxy_buffers*2）
proxy_busy_buffers_size    64k;
#设定缓存文件夹大小，大于这个值，将从upstream服务器传
proxy_temp_file_write_size 64k;
#反向代理缓存目录
proxy_cache_path /data/proxy/cache levels=1:2 keys_zone=cache_one:500m inactive=1d max_size=1g;
#levels=1:2 设置目录深度，第一层目录是1个字符，第2层是2个字符
#keys_zone:设置web缓存名称和内存缓存空间大小
#inactive:自动清除缓存文件时间。
#max_size:硬盘空间最大可使用值。
#指定临时缓存文件的存储路径(路径需和上面路径在同一分区)
proxy_temp_path /data/proxy/temp


### 服务配置
server {
    #侦听的80端口
    listen       80;
    server_name  localhost;
    location / {
        #代理设置
        proxy_pass   http://IP; 
        #重定向
        proxy_redirect off;
        #反向代理缓存设置命令(proxy_cache zone|off,默认关闭所以要设置)
        proxy_cache cache_one;
        #对不同的状态码缓存不同时间
        proxy_cache_valid 200 304 12h;
        #设置以什么样参数获取缓存文件名
        proxy_cache_key $host$uri$is_args$args;
        #后7端的Web服务器可以通过X-Forwarded-For获取用户真实IP
        proxy_set_header Host $host;
        #proxy_set_header X-Real-IP $remote_addr; 
        #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;   
        #文件过期时间控制
        expires    1d;
    }
    #配置手动清楚缓存(实现此功能需第三方模块 ngx_cache_purge)
    #http://www.123.com/2017/0316/17.html访问
    #http://www.123.com/purge/2017/0316/17.html清楚URL缓存
    location ~ /purge(/.*) {
        allow    127.0.0.1;
        deny    all;
        proxy_cache_purge    cache_one    $host$1$is_args$args;
    }
    #设置扩展名以.jsp、.php、.jspx结尾的动态应用程序不做缓存
    location ~.*\.(jsp|php|jspx)?$ { 
        proxy_set_header Host $host; 
        proxy_set_header X-Real-IP $remote_addr; 
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;   
        proxy_pass http://http://IP;
    }
```

Nignx状态监控：
```POWERSHELL
#Nginx运行状态，StubStatus模块获取Nginx自启动的工作状态（编译时要开启对应功能）
        #location /NginxStatus {
        #    #启用StubStatus的工作访问状态    
        #    stub_status    on;
        #    #指定StubStaus模块的访问日志文件
        #    access_log    logs/Nginxstatus.log;
        #    #Nginx认证机制（需Apache的htpasswd命令生成）
        #    #auth_basic    "NginxStatus";
        #    #用来认证的密码文件
        #    #auth_basic_user_file    ../htpasswd;    
        #}
访问：http://IP/NginxStatus(测试就不加密码验证相关) 
```

负载均衡:
```powershell
#负载均衡服务器池
upstream my_server_pool {
    #调度算法
    #1.轮循（默认）（weight轮循权值）
    #2.ip_hash：根据每个请求访问IP的hash结果分配。（会话保持）
    #3.fair:根据后端服务器响应时间最短请求。（upstream_fair模块）
    #4.url_hash:根据访问的url的hash结果分配。（需hash软件包）
    #参数：
    #down：表示不参与负载均衡
    #backup:备份服务器
    #max_fails:允许最大请求错误次数
    #fail_timeout:请求失败后暂停服务时间。
    server 192.168.1.109:80 weight=1 max_fails=2 fail_timeout=30;
    server 192.168.1.108:80 weight=2 max_fails=2 fail_timeout=30;
}
#负载均衡调用
server {
    ...
    location / {
    proxy_pass http://my_server_pool;
    }
}
```

URL重写:
```powershell
#根据不同的浏览器URL重写
if($http_user_agent ~ Firefox){
rewrite ^(.*)$  /firefox/$1 break; 
}
if($http_user_agent ~ MSIE){
rewrite ^(.*)$  /msie/$1 break; 
}

#实现域名跳转
location / {
    rewrite ^/(.*)$ https://web8.example.com$1 permanent;
}
```

IP限制:
```powershell
#限制IP访问
location / {
    deny 192.168.0.2；
    allow 192.168.0.0/24;
    allow 192.168.1.1;
    deny all;
}
```

配置完后，进入nginx默认安装目录（./sbin/nginx），找不到可以输入whereis ngingx进行范围查找；
进入安装目录后，可以pkill nginx杀死改进程，然后在nginx启动。
为了确保启动正常，可以lsof  -i:端口号，看端口是否被占用。

以上操作都正确后，如果还不能访问，可以telnet一下端口，看端口是否启动正常，如不能telnet，可能是防火墙问题，需开放该端口