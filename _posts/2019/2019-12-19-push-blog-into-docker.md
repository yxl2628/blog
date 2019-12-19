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
  - `github webhooks service（自动拉取github代码）`
  - `blog（博客）`
  - `theme（博客主题）`
  - `source(博客内容)`
  