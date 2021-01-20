---
title: ubuntu上搭建v2ray
date: 2019-11-16 08:52:13
categories: server
tags: vpn
---

### 前言

google cloud到期了，又撸了一个aws的免费服务器，不过用shadowsocks搭建的梯子，最近越来越不稳定了，改用v2ray搭建一个，不然很多技术问题，百度又查不到，google不不让查，很是苦恼。

说明（敲黑板）：身为前端，所有的类库都在国外，无数次npm install失败的时候的那种懊恼，是我想要搭个梯子的主要原因，本教程也只为了技术知识等正规用途，请勿使用此技术作违法的事哦

### 在线安装

1. 下载安装脚本：`curl -O https://install.direct/go.sh`

2. 安装v2ray：`sudo bash go.sh`

### 离线安装

> 由于该脚本默认执行时，需要访问国外的服务器下载v2ray-linux-64.zip文件，如果不走代理，这个过程会非常痛苦，而且有大概率会下载失败。我将go.sh以及目前（2019-12-05）为止最新的v2ray客户端，上传到了百度网盘，供大家下载：链接: https://pan.baidu.com/s/165ln8Wlmzza2dRca-46aog 提取码: am8h

如果对自己的网速有自信，也可以从官方网站下载安装包：[release](https://github.com/v2ray/v2ray-core/releases)

下载完成后，把go.sh与v2ray-linux-64.zip放在同一个文件夹，在终端下执行：`sudo bash go.sh --local ./v2ray-linux-64.zip`

也可以安装成功


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
          "path": "/ray"  # 与nginx中的路径保持一致
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

```
location /ray { # 与 V2Ray 配置中的 path 保持一致
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

这样，正常访问域名，既是一个正常的网站，但是一旦访问http://hostname/ray，即可伪装上网了

### 客户端

客户端下载：[官方下载](https://github.com/v2ray/v2ray-core/releases)

国内镜像下载：[国内镜像（tlanyan）](https://www.tlanyan.me/v2ray-clients-download/)

客户端配置像这样：

> 只保留了关键配置的说明，建议根据这个进行修改，而不是直接拷贝

```json
{
    "dns": {
        "servers": [
            "localhost"
        ]
    },
    "inbounds": [
        {
            "listen": "127.0.0.1",
            "port": 1081, # 可以在这里更改端口
            "protocol": "socks",
            "tag": "socksinbound",
            "settings": {
                "auth": "noauth",
                "udp": false,
                "ip": "127.0.0.1"
            }
        },
        {
            "listen": "127.0.0.1",
            "port": 8001, # 可以在这里更改端口
            "protocol": "http",
            "tag": "httpinbound",
            "settings": {
                "timeout": 0
            }
        }
    ],
    "outbounds": [
        {
            "tag": "direct",
            "protocol": "freedom",
            "settings": {}
        },
        {
            "sendThrough": "0.0.0.0",
            "mux": {
                "enabled": false,
                "concurrency": 8
            },
            "protocol": "vmess",
            "settings": {
                "vnext": [
                    {
                        "address": "？？？？", # 这里填写服务器的域名或者ip
                        "users": [
                            {
                                "id": "？？？？？", # 这里填写服务器上生成的uuid
                                "alterId": 64, # 这里填写服务器上的id
                                "security": "auto",
                                "level": 1
                            }
                        ],
                        "port": 443
                    }
                ]
            },
            "tag": "proxy server",
            "streamSettings": {
                "wsSettings": {
                    "path": "/v2ray", # 这里填写你服务器上nginx做的转发路径
                    "headers": {}
                },
                "quicSettings": {
                    "key": "",
                    "header": {
                        "type": "none"
                    },
                    "security": "none"
                },
                "tlsSettings": {
                    "allowInsecure": false,
                    "alpn": [
                        "http/1.1"
                    ],
                    "serverName": "？？？？", # 这里填写你服务器的ip或者域名
                    "allowInsecureCiphers": false
                },
                "httpSettings": {
                    "path": ""
                },
                "kcpSettings": {
                    "header": {
                        "type": "none"
                    },
                    "mtu": 1350,
                    "congestion": false,
                    "tti": 20,
                    "uplinkCapacity": 5,
                    "writeBufferSize": 1,
                    "readBufferSize": 1,
                    "downlinkCapacity": 20
                },
                "tcpSettings": {
                    "header": {
                        "type": "none"
                    }
                },
                "security": "tls",
                "network": "ws"
            }
        }
    ],
    "routing": {
        "name": "bypasscn_private",
        "domainStrategy": "IPIfNonMatch",
        "rules": [
            {
                "type": "field",
                "outboundTag": "direct",
                "ip": [
                    "geoip:private",
                    "geoip:cn"
                ]
            }
        ]
    },
    "log": {
        "error": "/var/log/v2ray/error.log", # 这里需要改称你自己想保存的日志路径
        "loglevel": "none", # 我觉着打印日志太占用空间，所以把日志关闭了，你也可以改成 info/warn/error
        "access": "/var/log/v2ray/access.log" # 这里需要改称你自己想保存的日志路径
    }
}

```

### 系统代理或者浏览器代理

客户端安装完成以后，如果是windows或者mac，那么已经可以直接使用代理了，但是linux上，需要自己去设置系统代理才行

- 系统代理：打开设置-->网络-->网络代理,方法选择手动,填写代理,最后点击应用到整个系统
- 浏览器代理：如果你只是想用浏览器上网，其他情况并不需要代理的话，那么推荐使用chrome的插件`SwitchyOmega`，就免去了来回切换代理的烦恼，而且也可以自己定义哪些网站通过代理。