---
title: docker介绍及使用
date: 2019-12-03 21:55:02
categories: server
tags: docker
---

### 前言

最近搭了一个梯子，整个过程搭建极其繁琐，当时就想，如果能我搭建完成这个环境之后，以后再搭建的时候，直接用就行了（因为在薅aws的羊毛-_-||，一年之后要用新的信用卡和账号注册(*^▽^*)）；
正巧最近工作中也遇到了问题，再本地搭建的一套演示环境，要部署到哈尔滨的客户现场（内网环境），从后端到前端，以及zabbix、grafana、ldap等等，安装过程及其的痛不欲生，明明就是一套已经搭建好的环境，为什么迁移过程如此复杂呢？
其实我是知道docker的，之前只是以为其类似虚拟机一样的技术，就是在服务器上启动一个虚拟机，但是对虚拟机不太有好感（虚拟机安装也不简单，还对多出来一堆莫名其妙的网口，以及内存占用及其庞大），也不是特别简单，所以就一直没深入了解，遇到迁移环境，快速部署的问题多了，才开始要了解，了解后，发现docker和我想象的大相径庭，实在是开发过程中的必备利器。

### Docker介绍

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。
容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

### Docker优点

1. 快速，一致地交付应用程序

Docker 允许开发人员使用您提供的应用程序或服务的本地容器在标准化环境中工作，从而简化了开发的生命周期。

容器非常适合持续集成和持续交付（CI / CD）工作流程，请考虑以下示例方案：

- 开发人员在本地编写代码，并使用 Docker 容器与同事共享他们的工作。

- 他们使用 Docker 将其应用程序推送到测试环境中，并执行自动或手动测试。

- 当开发人员发现错误时，他们可以在开发环境中对其进行修复，然后将其重新部署到测试环境中，以进行测试和验证。

- 测试完成后，将修补程序推送给生产环境，就像将更新的镜像推送到生产环境一样简单。

2. 响应式部署和扩展

Docker 是基于容器的平台，允许高度可移植的工作负载。Docker 容器可以在开发人员的本机上，数据中心的物理或虚拟机上，云服务上或混合环境中运行。
Docker 的可移植性和轻量级的特性，还可以使您轻松地完成动态管理的工作负担，并根据业务需求指示，实时扩展或拆除应用程序和服务。

3. 在同一硬件上运行更多工作负载

Docker 轻巧快速。它为基于虚拟机管理程序的虚拟机提供了可行、经济、高效的替代方案，因此您可以利用更多的计算能力来实现业务目标。Docker 非常适合于高密度环境以及中小型部署，而您可以用更少的资源做更多的事情。

### Docker架构

- 镜像（Image）：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。

- 容器（Container）：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

- 仓库（Repository）：仓库可看着一个代码控制中心，用来保存镜像。

### DockerFile

基础镜像：

> 安装 ubutu 18.04 LTS 版本，并将apt的源切换成了阿里源（如果你要将服务器部署到海外，可以将第二行注释掉）

```Dockerfile
FROM ubuntu:18.04
RUN cp /etc/apt/sources.list /etc/apt/sources.list.bak; \
  sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list; \
  sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list; \
  apt-get clean; \
  apt-get update
```

nodejs镜像：

> 在基础镜像的基础上安装nodejs

```Dockerfile
FROM ubuntu:18.04
RUN cp /etc/apt/sources.list /etc/apt/sources.list.bak; \
  sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list; \
  sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list; \
  apt-get clean; \
  apt-get update
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

ningx镜像：

> 在基础镜像的基础上安装了nginx；

```Dockerfile
FROM ubuntu:18.04
RUN cp /etc/apt/sources.list /etc/apt/sources.list.bak; \
  sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list; \
  sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list; \
  apt-get clean; \
  apt-get update
RUN apt-get install nginx -y
```

fe镜像：

> 大部分时候，我要么是需要单独的nginx，要么是需要node、npm、nginx全部，所以在单独写一个镜像出来，方便拷贝

```Dockerfile
FROM ubuntu:18.04
RUN cp /etc/apt/sources.list /etc/apt/sources.list.bak; \
  sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list; \
  sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list; \
  apt-get clean; \
  apt-get update
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
RUN apt-get install nginx -y
```


### 构建镜像

```bash
docker build -t basic .
```

### 删除镜像

```bash
docker images
docker rmi basic
```


### 启动容器

命令说明：`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

- -a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；

- -d: 后台运行容器，并返回容器ID；

- -i: 以交互模式运行容器，通常与 -t 同时使用；

- -P: 随机端口映射，容器内部端口随机映射到主机的高端口

- -p: 指定端口映射，格式为：主机(宿主)端口:容器端口

- -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；

- --name="nginx-lb": 为容器指定一个名称；

- --dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；

- --dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；

- -h "mars": 指定容器的hostname；

- -e username="ritchie": 设置环境变量；

- --env-file=[]: 从指定文件读入环境变量；

- --cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；

- -m :设置容器使用内存最大值；

- --net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；（Mac貌似不支持）

- --link=[]: 添加链接到另一个容器；

- --expose=[]: 开放一个端口或一组端口；

- --volume , -v: 绑定一个卷

实例：

```bash
# for all port
docker run -idt --name test-demo basic --net=host --restart=always
# for single port
docker run -idt --name test-demo basic -p 80:80 --restart=always basic
```
> -idt 实际上就是 -i -d -t,可以参照上面的说明，自行理解

### 进入容器

```bash
# bash 命令
docker exec -ti test-demo /bin/bash
```

### 停止容器

```bash
# look for docker
docker ps -a
# delete the current docker
docker stop test-demo
```

### 删除容器

```bash
# look for docker
docker ps -a
# delete the current docker
docker rm test-demo
```

### 国内加速站点

国内可用的源：

- Docker 官方中国区：https://registry.docker-cn.com
- 网易：http://hub-mirror.c.163.com
- 中国科技大学：https://docker.mirrors.ustc.edu.cn
- 阿里云：打开连接[https://cr.console.aliyun.com/#/accelerator](https://cr.console.aliyun.com/#/accelerator)拷贝您的专属加速器地址。

Mac上：

在任务栏点击 Docker for mac 应用图标 -> Perferences... -> Daemon -> Registry mirrors。在列表中填写加速器地址即可。修改完成之后，点击 Apply & Restart 按钮，Docker 就会重启并应用配置的镜像地址了。

Mac最新版本：
最新版本，修改的方法变了：在任务栏点击 Docker for mac 应用图标 -> Perferences -> Docker Engine，然后将国内源加入即可：
```
{
  "debug": true,
  "experimental": false,
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com"
  ]
}
```

Linux上：

如何更换：
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["上面的源随便填一个"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
