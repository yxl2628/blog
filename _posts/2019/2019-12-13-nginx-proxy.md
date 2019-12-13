---
title: nginx配置proxy_pass抓发的/路径问题
date: 2019-12-13 10:31:02
categories: server
tags: ngnix
---

### 前言

之前写过nginx的配置文档详解，当时拿着这个详解，基本上通吃所有常用配置了，不过今天遇到一个问题，折腾了好久，才发现，原来在`proxy_pass`上，还有隐藏的小坑，综合网上的资料，自己整理一份，以便加深记忆。

### 说明

在nginx中配置`proxy_pass`时，如果是按照^~匹配路径时,要注意`proxy_pass`后的url最后的`/`,当加上了`/`，相当于是绝对根路径，则nginx不会把location中匹配的路径部分代理走;如果没有`/`，则会把匹配的路径部分也给代理走。

#### proxy_pass不加/路径

```
location ^~/api/ {
  proxy_set_header Host $host;
  proxy_set_header  X-Real-IP        $remote_addr;
  proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
  proxy_set_header X-NginX-Proxy true;
  proxy_pass http://127.0.0.1:8080;
}
```

上面这种情况，如果请求的url是`http://yangxl.cn/api/login`，则会被代理成`http://127.0.0.1:8080/api/login`，这种也通常都是我们比较常用的情况，即配置的转发规则和实际后台服务器的应用前缀一致。

### proxy_pass加/路径

```
location ^~/api/ {
  proxy_set_header Host $host;
  proxy_set_header  X-Real-IP        $remote_addr;
  proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
  proxy_set_header X-NginX-Proxy true;
  proxy_pass http://127.0.0.1:8080/;
}
```

上面这种情况，如果请求的url是`http://yangxl.cn/api/login`，则会被代理成`http://127.0.0.1:8080/login`，最新用`egg.js`写了后台，发现没有应用名，然后我还按照之前的配制方法，结果死活访问不到，于是才有了这篇文章，简单来说，如果想要转发的时候，不加转发路径，就加`/`，如果想让转发路径和请求路径保持一致，就不要加`/`

#### proxy_pass不加/路径，同样实现加/的效果

那么，就不想要加`/`，但是又想要去掉前缀路径，有没有办法呢，是有的：

```
location ^~/api/ {
  proxy_set_header Host $host;
  proxy_set_header  X-Real-IP        $remote_addr;
  proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
  proxy_set_header X-NginX-Proxy true;
  rewrite /api/(.+)/1 break; 
  proxy_pass http://127.0.0.1:8080;
}
```
