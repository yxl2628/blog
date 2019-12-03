---
title: 自己使用的基础docker镜像
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

纯前端使用的环境：

```
FROM ubuntu:18.04
RUN apt update;apt install git curl nginx nodejs npm -y;
RUN npm install npm -g;npm install –g n stable;
```

做了各种优化的基础环境：

```
FROM ubuntu:18.04
RUN apt update;apt install git curl nano iputils-ping net-tools netcat zsh -y;chsh -s /bin/zsh
RUN sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)";cd ~;git clone https://github.com/zsh-users/zsh-syntax-highlighting.git;echo "source ~/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ~/.zshrc
```

### 构建镜像

```bash
docker build -t ubuntu .
```

### 启动镜像

```bash
# for all port
docker run -idt --name ubuntu --net=host --restart=always ubuntu
# for single port
docker run -idt --name ubuntu -p 80:80 --restart=always ubuntu
```

### 国内加速站点

在任务栏点击 Docker for mac 应用图标 -> Perferences... -> Daemon -> Registry mirrors。在列表中填写加速器地址即可。修改完成之后，点击 Apply & Restart 按钮，Docker 就会重启并应用配置的镜像地址了。

1. https://registry.docker-cn.com
2. http://hub-mirror.c.163.com
3. https://3laho3y3.mirror.aliyuncs.com
4. http://f1361db2.m.daocloud.io
5. https://mirror.ccs.tencentyun.com