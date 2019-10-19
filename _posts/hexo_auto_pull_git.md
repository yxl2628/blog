---
title: github提交hexo自动更新的博客设计
categories: server
tags: server nodejs git
---

### 前言
用了将近两年的github pages功能当做自己的个人博客，可是最近发现，在急着想要用已经总结好的技术资料的时候，网速成了制约，github pages的打开速度，在国内有时候实在是惨不忍睹，由此萌生了把博客搬家回国内的想法。

### OKR
> 虽然只是一个小小的迁移工作，也不妨制定一个OKR
#### OKR-1
    O（目标）：尽可能的采用小的成本来完成

    KR（关键结果）：使用阿里云最小最低配置的服务器来完成博客搭建
    
  目前阿里云最低方案为：512M内存，双核且5%性能的VPS
  ![aliyun](/images/server/aliyun-server.jpg)


#### OKR-2
    O（目标）：编写博客不能造成除写博客之外的工作量

    KR（关键结果）：除编写过程，其它过程全部采用自动化方案来实现

### 技术架构
> 技术架构图设计如下：
![aliyun](/images/server/auto-blog.png)

关键技术：
1. 将hexo的源文件目录设置成github仓库，这样就能将博客通过github管理起来
2. 用nginx将hexo的发布目录监听起来，这样hexo每次构建之后，ningx都可以及时访问
3. github提供的webhooks是将所有流程串联起来的关键技术
4. 自己用nodejs写一个简单的服务器，用来监听git webhooks（为了防止端口暴露，将此服务通过nginx proxy代理）
5. 使用nodejs的shelljs来调用linux的shell命令

总结：编写博客-->提交到github-->github通知服务器-->服务器接收到通知-->调用git pull，将最新博客拉取到source-->拉取成功后，调用hexo generate生成静态页面到public-->ningx监听public，用户访问到最新资源

### 相关源码
> 待补充