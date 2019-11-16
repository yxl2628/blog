---
title: ubuntu上搭建v2ray
date: 2019-11-16 08:52:13
categories: server
tags: linux vpn
---

### 前言

google cloud到期了，又撸了一个aws的免费服务器，不过用shadowsocks搭建的梯子，最近越来越不稳定了，改用v2ray搭建一个，不然很多技术问题，百度又查不到，google不不让查，很是苦恼。

说明（敲黑板）：身为前端，所有的类库都在国外，无数次npm install失败的时候的那种懊恼，是我想要搭个梯子的主要原因，本教程也只为了技术知识等正规用途，请勿使用此技术作违法的事哦

### 安装

1. 下载安装脚本：`curl -O https://install.direct/go.sh`

2. 安装v2ray：`sudo bash go.sh`

3. 相关命令

```shell
## 启动
systemctl start v2ray

## 停止
systemctl stop v2ray

## 重启
systemctl restart v2ray

## 开机自启
systemctl enable v2ray
```

4. 配置文件，默认存放路径为：`/etc/v2ray/config.json`

### 高级技巧：流量伪装

网上列举了几种抗封锁的v2ray配置组合：

- vmess
- vmess + http
- TLS + http2
- websocket + TLS + nginx/caddy

第四种的抗封锁效果最好，可谓是最优配置，会把流量伪装成其它应用的SSL安全业务类型，第四种伪装的好处就是：

- 从外面看完全属于通过443端口访问正常的HTTPS网站
- websocket的path（比如教程中的/ray）被TLS加密不能被探测到
- TLS不是伪装，不是混淆，是真正的HTTPS，用途合理永远不能被封锁

v2ray配置： `sudo vi /etc/v2ray/config.json` 和 `sudo systemctl restart v2ray`

```json
{
  "log": {
    "loglevel": "warning",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
   },
  "inbounds": [{
    "port": 12345,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "xxxxx", # 可以使用v2ctl uuid生成
          "level": 1,
          "alterId": 64
        }
      ]
    },
    "streamSettings": {     # 载体配置段，设置为websocket
        "network": "ws",
        "wsSettings": {
          "path": "/v2ray"  # 与nginx中的路径保持一致
        }
      },
    "listen": "127.0.0.1" # 出于安全考虑，建议只接受本地链接
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```
注意： json不接受“#”注释，上述配置，在实际使用时，应该删除

nginx配置：

```conf
location /v2ray { # 与 V2Ray 配置中的 path 保持一致
  proxy_redirect off;
  proxy_pass http://127.0.0.1:12345; # 假设v2ray的监听地址是12345
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
  proxy_set_header Host $host;
  # Show real IP in v2ray access.log
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

这样，正常访问域名，既是一个正常的网站，但是一旦访问http://hostname/aws-v2ray，即可伪装上网了

### 客户端

客户端下载：[官方下载](https://github.com/v2ray/v2ray-core/releases)

国内镜像下载：[国内镜像（tlanyan）](https://www.tlanyan.me/v2ray-clients-download/)

客户端配置像这样：

> 只保留了关键配置的说明，建议根据这个进行修改，而不是直接拷贝

```json
{
    "outbounds": [
        {
            "protocol": "vmess",
            "settings": {
                "vnext": [
                    {
                        "address": "你的服务器ip或域名",
                        "users": [
                            {
                                "id": "和服务器上的uuid保持一致",
                                "alterId": 64, #和服务器上的id保持一致
                                "security": "auto",
                                "level": 1
                            }
                        ],
                        "port": 443 #注意，这里填的是443，不是v2ray配置的端口号12345，因为我们要做流量伪装，这里填的是实际网站的https端口
                    }
                ]
            },
            "streamSettings": {
                "wsSettings": {
                    "path": "v2ray" #这里就是nginx配置的转发的path
                },
                "tlsSettings": {
                    "serverName": "这里填写你的网站域名"
                },
                "security": "tls",
                "network": "ws"
            }
        }
    ]
}
```