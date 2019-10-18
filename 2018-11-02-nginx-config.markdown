---
layout: post
title: nginx config配置
date: 2018-11-2 11:35:56
tags: ngnix
---

```
#运行用户，默认即是nginx，可以不进行设置
user nginx;
#Nginx进程，一般设置为和CPU核数一样
worker_processes 1;
#错误日志存放目录
error_log /var/log/nginx/error.log warn;
#进程pid存放位置
pid /var/run/nginx.pid;

events
{
  worker_connections 51200; # 每个进程最大连接数（最大连接=连接数×进程数）
  accept_mutex on; # 将使每个可用的worker进程逐个接受新连接
  multi_accept on; # 使得NGINX worker能够在获得新连接的通知时尽可能多的接受连接
  use epoll; # 使用epoll（linux2.6的高性能方式）
}

http
{
  include /etc/nginx/mime.types; #文件扩展名与类型映射表
  default_type application/octet-stream; #默认文件类型

  limit_req_zone $binary_remote_addr zone=one:10m rate=20r/s; # zone:one:10m 表示一个内存区域大小为10m,并且设定了名称为one；rate=20r/s 表示请求的速率是1秒20个请求
  limit_conn_zone $binary_remote_addr zone=addr:10m; # 请求连接数限制 参数含义同上

  #设置日志模式
  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
  '$status $body_bytes_sent "$http_referer" '
  '"$http_user_agent" "$http_x_forwarded_for"';

  access_log /var/log/nginx/access.log main; #nginx访问日志存放位置

  sendfile on; #开启高效传输模式
  #tcp_nopush     on;    #减少网络报文段的数量

  keepalive_timeout 65; #保持连接的时间，也叫超时时间

  #gzip  on;  # 开启gzip压缩
  gzip_buffers 32 4k; # 这里表示每压缩32个包，每个包4k大小，就向客户端发送
  gzip_comp_level 6; # 这里表示压缩级别，可以是0到9中的任一个，级别越高，压缩就越小，节省了带宽资源，但同时也消耗CPU资源，所以一般折中为6
  gzip_min_length 200; # 这里表示如果文件小于200个字节，就不用压缩，因为没有意义，本来就很小
  gzip_types text/css text/xml application/javascript; # 哪些类型的文件要压缩，text/html是默认不需要写，/etc/nginx/mime.types 文件中有所有可以压缩的文件类型
  gzip_vary on; # 可以不写，表示我在传送数据时，给客户端说明我使用了gzip压缩

  # include /etc/nginx/conf.d/*.conf; #包含的子配置项位置和文件

  server
  {
    listen 80; #配置监听端口
    server_name localhost; //配置域名

    #access_log  /var/log/nginx/host.access.log  main;

    location /
    {
      limit_req zone=one burst=5 nodelay; # 链接请求数限制（request是指请求，即http请求，注意，tcp连接是有状态的，而构建在tcp之上的http却是无状态的协议）
      limit_conn addr 5; # 链接数限制（connection是连接，即常说的tcp连接，通过三次握手而建立的一个完整状态机）
      root /usr/share/nginx/html; #服务默认启动目录
      index index.html index.htm; #默认访问文件
    }

    #error_page  404              /404.html;   # 配置404页面

    # redirect server error pages to the static page /50x.html
    #
    error_page 500 502 503 504 /50x.html; #错误状态码的显示页面，配置后需要重启
    location = /50x.html
    {
      root /usr/share/nginx/html;
    }

    # 通过下面的三个缓存设置，可以最大程度的优化浏览器访问网站的性能（注意，这么做的前提是要保证强制缓存的资源是通过类似hash方式命名，否则，你会发现自己的更新不起作用）
    location ~*.*\.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm)$ # 表示对所有媒体资源文件做强制缓存
    {
      expires max;
    }
    location ~ .*\.(?:js|css)$ # 表示对所有js、css文件强制缓存
    {
      expires max;
    }
    location ~ .*\.(?:htm|html)$ # 声明html文件不允许浏览器缓存
    {
      add_header Cache-Control "private, no-store, no-cache, must-revalidate, proxy-revalidate";
    }
    # 虚拟路径映射，用于将静态数据隔离出来，和应用文件部署分离
    location ^~ /alias
    {
      if ( $request_method = OPTIONS )
      {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept, Access-Control-Allow-Headers, Authorization';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
        return 200;
      }
      add_header 'Access-Control-Allow-Origin' '*';
      add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept, Access-Control-Allow-Headers, Authorization';
      add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
      alias /home/data/;
    }
    # 反向代理（划重点），前后端分离项目中，用于解决跨域问题
    location /proxy
    {
      if ( $request_method = OPTIONS )
      {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept, Access-Control-Allow-Headers, Authorization';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
        return 200;
      }
      add_header 'Access-Control-Allow-Origin' '*';
      add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept, Access-Control-Allow-Headers, Authorization';
      add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
      proxy_pass http://127.0.0.1:8080/tomcat/api;
      # 注：nginx默认转发配置为：$http_host；不会改变请求头的值，这种情况下，静态文件是无法访问的，会导致404；需要加上这个配置，才能让nginx代理转发静态文件资源
      proxy_set_header Host $proxy_host;
    }
  }

  # server # 这种配置方式，一般用在配置https时使用，可以让http链接自动跳转到https
  # {
  #   listen 80;
  #   server_name www.xxxx.com;
  #   rewrite ^(.*) https://$server_name$1 permanent;
  # }

  # SSL 配置
  server
  {
    listen 443; # 监听端口，http默认端口是80，https默认端口即为443
    access_log logs/access.log main;
    server_name xxxx.com;
    ssl on; # 开启ssl
    root html;
    index index.html index.htm;
    ssl_certificate ../cert/xxxx.pem; # 配置证书位置
    ssl_certificate_key ../cert/xxxx.key; # 配置私钥位置
    ssl_session_timeout 5m; # 链接超时时间
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; # 密码加密方式
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # 指定密码为openssl支持的格式
    ssl_prefer_server_ciphers on; # 依赖SSLv3和TLSv1协议的服务器密码将优先于客户端密码
    location /
    {
      root html;
      index index.html index.htm;
    }
  }
}
```
