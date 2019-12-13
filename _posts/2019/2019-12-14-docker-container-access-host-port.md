---
title: docker容器内访问宿主机服务器的方法
date: 2019-12-14 17:45:25
categories: server
tags: docker
---

### 前言

Docker内需要访问本机的数据库，如何访问。使用127.0.0.1肯定是不行的，因为这个在Docker容器里面指的是容器本身。所以，需要走别动渠道进行解决。

### 解决方案

#### DockerFile

```
RUN /sbin/ip route|awk '/default/ { print  $3,"\tdockerhost" }' >> /etc/hosts
```

#### RunTime

```
(may not use) docker run --add-host dockerhost:`/sbin/ip route|awk '/default/ { print  $3}'` [my container]
(useful) docker run --add-host=dockerhost:`docker network inspect  --format='{{range .IPAM.Config}}{{.Gateway}}{{end}}' bridge` [IMAGE]
```

#### Docker for Mac (17.12+)

```
docker.for.mac.host.internal
```

#### Linux

```
＃ Solution 1
/sbin/ip route|awk '/default/ { print $3 }'
docker run --add-host dockerhost:`/sbin/ip route|awk '/default/ { print  $3}'` [my container]
＃ Solution 2
-e "DOCKER_HOST=$(ip -4 addr show docker0 | grep -Po 'inet \K[\d.]+')"
```

