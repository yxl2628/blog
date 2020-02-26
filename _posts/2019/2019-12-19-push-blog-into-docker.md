---
title: docker实战：将博客网站的部署打入docker镜像中
date: 2019-12-19 17:37:02
categories: server
tags: docker
---

### 前言

经过前几天的学习，基本上掌握了docker的使用方法，但是具体能不能实际应用上，应用的好处是什么？还需要实战来检验，正好我的blog刚刚搭建完成，计划拿它来做实战（原计划用搭梯子来做镜像，不过梯子其实本身有点简单，涉及到的内容有点少，所以改成用blog来写这篇blog）

### 分析

需要导入镜像的内容有：
  - [github webhooks service](https://github.com/yxl2628/auto-public-hexo.git)（自动拉取github代码）
  - [blog](https://hexo.io/)（博客）
  - [theme](https://github.com/yxl2628/hexo-theme-Chic)（博客主题）
  - [source](https://github.com/yxl2628/blog)(博客内容)
  - nginx(用于做网站代理)

### 搭建基础镜像

#### 1.选择ubuntu 18.04 LTS 为基础镜像

```Dockerfile
# base image
FROM ubuntu:18.04
```

#### 2.将apt源更换为阿里源

> 国内访问的话，镜像源有点慢，这里更换成阿里的，当然，如果服务器本身部署在国外，那么此步略过

```Dockerfile
# base image
# change apt sources
RUN cp /etc/apt/sources.list /etc/apt/sources.list.bak; \
  sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list; \
  apt-get clean; \
  apt-get update
```

#### 3.安装最新版nodejs，并安装cnpm

> 道理是一样的道理，国内npm源也不是很快，所以安装了淘宝npm，如果在海外，可以不安装，也可以不用，不用的话，就是将之后的cnpn install改为npm install即可

```Dockerfile
# base image
# change apt sources
# install nodejs & cnpm
RUN apt-get install curl wget -y \
  && cd /opt \
  && lastVersion="node-${VERSION:-$(wget -qO- https://nodejs.org/dist/latest-v12.x/ | sed -nE 's|.*>node-(.*)\-linux-x64.tar.gz</a>.*|\1|p')}-linux-x64" \
  && curl "https://nodejs.org/dist/latest-v12.x/${lastVersion}.tar.gz" > "node-latest.tar.gz" \
  && cd /opt && tar zxvf node-latest.tar.gz \
  && mv "${lastVersion}" nodejs \
  && ln -s /opt/nodejs/bin/node /usr/local/bin/node \
  && ln -s /opt/nodejs/bin/npm /usr/local/bin/npm \
  && npm install -g cnpm --registry=https://registry.npm.taobao.org \
  && ln -s /opt/nodejs/bin/cnpm /usr/local/bin/cnpm \
  && rm -rf node-latest.tar.gz
```

#### 4.安装nginx、hexo

```Dockerfile
# base image
# change apt sources
# install nodejs & cnpm 
# install nginx & hexo & clone git
RUN apt-get install nginx git -y \
  && cnpm install hexo-cli -g \
  && ln -s /opt/nodejs/bin/hexo /usr/local/bin/hexo \
  && cd /opt/ \
  && git clone https://github.com/yxl2628/hexo.git blog \
  && cd /opt/blog \
  && cnpm install \
  && git clone https://github.com/yxl2628/blog.git source \
  && cd /opt \
  && git clone https://github.com/yxl2628/auto-public-hexo.git \
  && cd /opt/auto-public-hexo \
  && cnpm install
```

### 配置nginx

上面的镜像搭好，但是各个功能还比较分散，接下来用nginx来串联起各个功能

nginx主要做两件事：

1. 博客网站web服务器
2. webhooks servie 代理转发

```conf
http {
  server {
    listen 80;
    server_name www.yangxl.cn;
    rewrite ^(.*) https://$server_name$1 permanent;
  }

  server {
    listen 443 ssl http2 default_server;
    listen [::]:443 ssl http2 default_server;
    server_name www.yangxl.cn;
    #root         /usr/share/nginx/html;
    root	/opt/blog/public;

    ssl_certificate "cert/2931975_www.yangxl.cn.pem";
    ssl_certificate_key "cert/2931975_www.yangxl.cn.key";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout 10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    location /git-hooks {
      if ( $request_method = OPTIONS ) {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept, Access-Control-Allow-Headers, Authorization';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
        return 200;
      }
      add_header 'Access-Control-Allow-Origin' '*';
      add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept, Access-Control-Allow-Headers, Authorization';
      add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
      proxy_pass http://127.0.0.1:8888/git-hooks;
      proxy_set_header Host $proxy_host;
    }
  }
}
```

### 镜像默认启动脚本

增加一个镜像的默认启动脚本，让镜像启动时自动执行以下事项

1. 更新git仓库最新代码

2. 安装更新node_modules

3. 启动nginx（这里把nginx脚本外挂到宿主机，方便修改）

4. 启动webhooks service

```bash
#!/bin/sh
# add the host ip into the container hosts
/sbin/ip route|awk '/default/ { print  $3,"\tdockerhost" }' >> /etc/hosts
# update the code lasetst
echo "==================update hexo======================"
cd /opt/blog
git fetch --all
git reset --hard origin/master
echo "==================update githooks server======================"
cd /opt/auto-public-hexo
git fetch --all
git reset --hard origin/master
# generate html
echo "==================update blog======================"
cd /opt/blog/source
git fetch --all
git reset --hard origin/master
hexo g
# start nginx
echo "==================start nginx======================"
/usr/sbin/nginx -c /opt/nginx/nginx.conf
# start auto-publish-server
echo "==================start githooks server======================"
cd /opt/auto-public-hexo
node src/server.js
```

### Dockerfile增加启动配置

上面的dockerfile为了清晰，只保留了说明部分的代码，下面的dockerfile是完整的：

```Dockerfile
# base image
FROM ubuntu:18.04
# change apt sources
RUN cp /etc/apt/sources.list /etc/apt/sources.list.bak; \
  sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list; \
  apt-get clean; \
  apt-get update
# install nodejs & cnpm 
RUN apt-get install curl wget -y \
  && cd /opt \
  && lastVersion="node-${VERSION:-$(wget -qO- https://nodejs.org/dist/latest-v12.x/ | sed -nE 's|.*>node-(.*)\-linux-x64.tar.gz</a>.*|\1|p')}-linux-x64" \
  && curl "https://nodejs.org/dist/latest-v12.x/${lastVersion}.tar.gz" > "node-latest.tar.gz" \
  && cd /opt && tar zxvf node-latest.tar.gz \
  && mv "${lastVersion}" nodejs \
  && ln -s /opt/nodejs/bin/node /usr/local/bin/node \
  && ln -s /opt/nodejs/bin/npm /usr/local/bin/npm \
  && npm install -g cnpm --registry=https://registry.npm.taobao.org \
  && ln -s /opt/nodejs/bin/cnpm /usr/local/bin/cnpm \
  && rm -rf node-latest.tar.gz
# install nginx & hexo & clone git
RUN apt-get install nginx git -y \
  && cnpm install hexo-cli -g \
  && ln -s /opt/nodejs/bin/hexo /usr/local/bin/hexo \
  && cd /opt/ \
  && git clone https://github.com/yxl2628/hexo.git blog \
  && cd /opt/blog \
  && cnpm install \
  && git clone https://github.com/yxl2628/blog.git source \
  && cd /opt \
  && git clone https://github.com/yxl2628/auto-public-hexo.git \
  && cd /opt/auto-public-hexo \
  && cnpm install
# start
ENTRYPOINT ["/bin/sh", "/opt/blog/startup.sh"]
```

### 构建镜像

```bash
docker build -t blog . 
```

### 运行容器

```bash
docker run -idt --name blog -p 80:80 -p 443:443 -v /opt/private/blog/nginx:/opt/nginx  --restart=always blog
```

> 上面挂在的文件路径`/opt/private/blog/nginx`，是我的nginx.conf配置和tls证书存放的位置，小伙伴们可以根据需要，自己修改位置

### 更新git

```bash
docker restart blog
```

### 进入容器

```bash
docker exec -ti blog /bin/bash
```

### 手动更新脚本

如果觉着重启docker，更新步骤太多，也可以用这个重启脚本

```shell
echo "===================ps all node=================="
ps -ef|grep node
echo "==================kill all node================="
ps -ef | grep node | awk '{print $2}' | xargs kill -9
echo "==================show the kill result================="
ps -ef|grep node
echo "==================generate html================="
cd /opt/blog/source
git fetch --all
git reset --hard origin/master
cd /opt/blog
hexo g
echo "================restart the node================"
cd /opt/auto-public-hexo
node src/server.js
```