---
title: 跨端（小程序/H5/App）前端开发框架调研和应用
date: 2021-05-13 17:32:00
categories: javascript
tags: fe
---

### 跨端的前端方案的需求

1. 满足各种主流小程序（微信/支付宝/百度/头条）、应用商店的快应用、APP端的封装的要求
2. 方案成熟，社区活跃，遇到问题晚上可查方案以及官方能够回复解决
3. 开源的团队要相对稳定，最好不要是个人开发者

### 主流支持跨端的前端框架广泛调研

- **[Taro](https://taro.jd.com/)**

  京东凹凸实验室发起的开源项目，2.0版本是基于React，3.x进行了重构，可以支持React/Vue/Vue3.0等框架

  - 多端支持（9端支持）
  - 双向转换（支持微信小程序和Taro的互相转换）
  - Taro Doctor编译检查方案
  - 京东小程序可以可视化搭建
  - 有自己的UI组件库，[Taro UI](https://taro-ui.jd.com/)

- **[uni-app](https://uniapp.dcloud.io/)**

  DCloud团队开发的基于Vue开发的跨平台框架，目前是中国开发者最多的框架，而且有自研的IDE工具

  - 多端支持（10端支持）
  - 国内开发者数量最多
  - 具备自己的开发IDE，HbuildX，开发效率高
  - 有[插件市场](https://ext.dcloud.net.cn/)，可以方便接入

- **[chameleon](https://cml.js.org/)**

  滴滴开源的跨平台开发框架，基于MVVM架构的自研多态协议，编写多段代码体验最好的框架

  - 独创的`前端中台服务`，开发效率高
  - 独创的多态协议，真正的一套代码实现多端（`taro`和`uni-app`需要条件编译）
  - 多端高度统一，无需关注各端文档
  - 渐进式接入，方便原有功能迁移

- [wepy](https://wepyjs.gitee.io/wepy-docs/)

  微信小程序最早开源的专门用来开发微信小程序的框架，比原生开发效率高，而且可以和原生组件混编，wepy2.0计划开放多端支持，但已经处于alpha很长时间，疑似停止维护

- [**mpvue**](http://mpvue.com/)

  美团开源的基于Vue的用来开发微信小程序的框架，和wepy一样，是为了解决微信小程序初期不支持自定义组件而诞生的，目前已停止更新

- **[kbone](https://developers.weixin.qq.com/miniprogram/dev/extended/kbone/)**

  微信小程序专门为了解决微信小程序和web端同构的框架，支持多种前端框架如：vue、react、preact等，为了更专注于小程序开发，用来替代wepy的框架，但是因为官方有过放弃wepy的行为，对kbone的可持续性存疑

#### 对比一：跨端支持度

|框架 |微信小程序 |支付宝小程序 |百度小程序 |头条小程序 |H5 |App |
|:-: |:-: |:-: |:-: |:-: |:-: |:-: |
|Taro |⭕ |⭕ |⭕ |⭕ |⭕ |⭕ |
|uni-app |⭕ |⭕ |⭕ |⭕ |⭕ |⭕ |
|chameleon |⭕ |⭕ |⭕ |⭕ |⭕ |⭕ |
|wepy |⭕ |❌ |❌ |❌ |❌ |❌ |
|mpvue |⭕ |❌ |❌ |❌ |❌ |❌ |
|kbone |⭕ |❌ |❌ |❌ |⭕ |❌ |

由上表可知，支持度的对比结果为：`Taro`、`uni-app`、`chameleon` **>** `kbone` **>** `mpvue`、`wepy`

#### 对比二：开发效率

| 功能 | wepy | mpvue | taro | uni-app | chameleon |
| :-: | :-: | :-: | :-: | :-: | :-: |
|技术栈  |类Vue  |类Vue  |React/Vue  |Vue  |类Vue  |
|学习成本  |⭕ |❌ |❌ |❌ |⭕ |
|IDE/插件  |❌	|❌ |`Visual Studio Code`	|⭕ |`Visual Studio Code`、`WebStorm`、`Sublime`、`Atom`	|
|语法校验 |❌ |❌ |ESlint规则 |IDE支持 |自研 |
|TypeScript |⭕ |⭕ |⭕ |⭕ |⭕ |
|自动补全 |API |❌ |API + JSX |IDE支持 |无 |
|样式 |sass、less、stylus |sass、less、stylus |sass、less、stylus、css modules |sass、less、stylus |sass、less、stylus |

1. 开发工具的效率

`uni-app`具有自己研发的HBuilderX开发工具，因此开发效率上，`uni-app`一骑绝尘，`taro`提供了`Visual Studio Code`插件，`chameleon`提供了`Visual Studio Code`、`WebStorm`、`Sublime`、`Atom` 插件

2. 组件库/工具库/Demo

| 功能 | wepy | mpvue | taro | uni-app | chameleon |
| :-: | :-: | :-: | :-: | :-: | :-: |
|自研组件库 |❌ |❌ |⭕ |⭕ |⭕ |
|第三方组件 |丰富 |丰富 |较丰富 |丰富 |较少 |
|第三方工具 |丰富 |较丰富 |较丰富 |丰富|较少 |
|Demo |丰富 |较丰富 |较丰富 |较丰富 |较少 |
|状态管理 |Redux |Vuex |Redux/Vuex/Dva/MobX |Vuex |Vuex |

综合对比结果：`uni-app` **>** `Taro` **>** `chameleon` **>** `mpvue` **>** `wepy`

#### 对比三：渲染性能

> dcloud团队写了一份多个框架的[测试代码](https://github.com/dcloudio/test-framework)，分别使用微信原生版(kbone)、wepy版、mpvue版、taro版、uni-app版、chalemeon版，各自开发一个仿微博小程序首页的复杂长列表，支持下拉刷新、上拉翻页、点赞功能。

| 功能 | 微信原生 | wepy | mpvue | taro | uni-app | chameleon |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: |
|200  |770	|625	|969	|752	|641	|1261	|
|400  |876  |781  |4493	|974 |741  |1970  |
|600  |1111 |-  |-  |1250	|910  |2917	  |
|800  |1406 |-  |-  |1547	|1113	|4040 |
|1000 |1690 |-  |-  |1878	|1321	|5196 |

- **说明1：**

以400条微博列表为例，从页面空列表开始，每隔1秒触发一次上拉加载（新增20条微博），记录单次耗时，触发20次后停止（页面达到400条微博），计算这20次的平均耗时，结果微信原生在这20次 `触发上拉 -> 渲染完成` 的平均耗时为876毫秒，最快的`uni-app`是741毫秒，最慢的mpvue是4493毫秒

- **说明2：**

`mpvue`、`wepy`是将用户编写的`Vue`组件，编译为`WXML`中的模板（template)，但是当页面dom节点增加时，开销会变得非常大，因此在600+条数场列表时，会报错停止渲染

- **说明3：**

`taro`和`uni-app`是基于`Vue`和`React`的开发框架，本身就有差异化diff计算，仅传递更新的数据，因此在不进行任何优化的情况下，性能会略高于原生（1000条时，框架本身的优化效果已经不明显）

由上表得到的对比结果为：`uni-app` **>** `taro` **>** `微信原生开发` **>** `chameleon` **>** `wepy` **>** `mpvue`

#### 对比四：社区活跃度

| 功能 | wepy | mpvue | taro | uni-app | chameleon |
| :-: | :-: | :-: | :-: | :-: | :-: |
|地址 |[地址](https://github.com/Tencent/wepy)	|[地址](https://github.com/Meituan-Dianping/mpvue)	|[地址](https://github.com/NervJS/taro)	|[地址](https://github.com/dcloudio/uni-app)	|[地址](https://github.com/didi/chameleon) |
|GitHub Star  |21.3k	|20.3K	|28.8K	|31.2K	|8.4K |
|GitHub Issue  |344/1777  |419/1264	|674/6546	|800/1589	|147/274 |
|开发者人数  |169	|135	|318	|146	|7 |
|开发者社区  |❌ |❌ |❌ |⭕ |❌ |

> 说明：uni-app的github活跃度低是因为他有自建的[开发者社区](https://ask.dcloud.net.cn/explore/)，因此活跃度基本上都转移到了该社区

### 选择分析

#### 权重一：跨端能力

  权重最高，如果对跨端有需求，那么从跨端支持度上来说，只能选择：`uni-app`、`Taro`、`chameleon`，`wepy`、`mpvue`首先被排除了


#### 权重二：组件库（案例）丰富程度

  权重次之，案例多说明使用者多，踩坑的人也多，相应的解决方案也多，组件库丰富，开发效率提升，毕竟不可能所有的组件都自己开发，`chameleon`因最晚发布，虽然前景可能很好，但明显不具备生产环境使用条件，`wepy`、`mpvue`虽然案例也较多，但是已被权重一排除

#### 权重三：技术栈-学习成本

  团队技术栈是Rect，那么使用`Taro`无疑是最好的选择
  如果团队的的技术栈为Vue，那么`uni-app`即为最佳答案

因此，最终的选型结果为：`Taro`和`uni-app`

### Taro踩坑指南

因OTT团队主要框架为React（Vue在移动端也有使用），所以开发时，第一选择使用了`Taro`，代码仓库地址：[智慧社区](https://gitlab.startimes.me/starott/ottapplication/tree/release-2.49.2/mobile/community-h5),在开发过程中，遇到了如下的几个问题：

#### Taro在H5上缺少TabBar，需要自己写代码适配

```javascript
// NavBar.h5.tsx
import React from 'react'
import Taro from '@tarojs/taro'
import { View } from '@tarojs/components'
import { AtNavBar } from 'taro-ui'
import './index.less'

export default (props: any) => {
  const back = () => {
    const pages = Taro.getCurrentPages();
    let canBack = true;
    if (pages.length === 1) {
      canBack = false;
    }
    if (pages[pages.length - 2] && pages[pages.length - 2].path.indexOf('/pages/auth/index') >= 0) {
      canBack = false;
    }
    canBack ? Taro.navigateBack() : Taro.switchTab({ url: '/pages/home/index' })
  }
  return (
    <View>
      <AtNavBar customStyle={{ opacity: 0 }} leftIconType="chevron-left" color="#fff" onClickLeftIcon={back} {...props} />
      <AtNavBar className="fix-nav" leftIconType="chevron-left" color="#fff" onClickLeftIcon={back} {...props} />
    </View>
  )
}
// NavBar.tsx
export default (props: any) => {
  return null
}
// Detail.tsx
import React, { useState, useEffect } from 'react'
import Taro from '@tarojs/taro'
import dayjs from 'dayjs'
import { View, Text, RichText, Swiper, SwiperItem, Image } from '@tarojs/components'
import NavBar from '../../components/NavBar'
import { getInformation } from '../../service'
import './index.less'

export default () => {
  const [information, setInformation] = useState(null)

  useEffect(() => {
    const { id } = Taro.getCurrentInstance().router.params
    fetchData(id)
  }, [])

  const fetchData = (id: string) => {
    // code
  }

  if (!information) {
    return null
  }
  const { name, time, description, posters } = information
  
  return (
    <View>
      <NavBar title="" />
      <View className="detail">
        <View className="detail-title"><Text>{name}</Text></View>
        <View className="detail-date"><Text>{time}</Text></View>
        <View className="detail-content">
          <RichText nodes={description} />
        </View>
      </View>
    </View>
  )
}
```

#### H5上tabbar消失

这个问题，提交了issue，但是官方修复的速度很慢，等了很久，依然没有修复，可能`Taro`团队对H5不积极，这里使用了hack的方式，来解决，代码如下：

```javascript
// app.ts
import { Component } from 'react'
import Taro from '@tarojs/taro'
import 'taro-ui/dist/style/index.scss'
import './app.less'

class App extends Component {
  async componentDidMount () {
    // TODO: hack tabbar error for h5
    if (process.env.TARO_ENV === 'h5') {
      const hash = document.URL.split('#')[1].split('?')[0]
      const index = ['/pages/home/index', '/pages/notice/index', '/pages/service/index', '/pages/mine/index'].indexOf(hash)
      if (index >= 0) {
        Taro.switchTab({ url: hash })
      }
    }
  }
  render () {
    return this.props.children
  }
}

export default App
```

#### H5端下拉刷新无效，滚动加载无效

官方文档明确指出了不支持H5端的上拉刷新以及下拉加载，需要自己编码实现：

```javascript
// list.jsx
import React, { useEffect, useState } from 'react';
import Taro from '@tarojs/taro';

export default () => {

  const [startX, setStartX] = useState(0);
  const [startY, setStartY] = useState(0);
  const [endX, setEndX] = useState(0);
  const [endY, setEndY] = useState(0);

  useEffect(() => {
    // code here
  }, []);

  const touchStart: any = (e: { touches: any[]; }) => {
    const touch = e.touches[0]
    setStartX(touch.pageX);
    setStartY(touch.pageY);
  }

  const touchMove: any = (e: { touches: any[]; }) => {
    const touch = e.touches[0]
    const deltaX = touch.pageX - startX;
    const deltaY = touch.pageY - startY;
    setEndX(deltaX);
    setEndY(deltaY);
  }

  const touchEnd: any = () => {
    if (endY < -100) { // 向下滑动
      setEndY(0);
      setEndX(0);
      // code here
    }
    if (endY > 100) {
      setEndY(0);
      setEndX(0);
      // code here
    }
    if (endX < -100) {
      // 向左滑动
    }
    if (endX > 100) {
      // 向右滑动
    }
  }

  return <View onTouchStart={touchStart} onTouchMove={touchMove} onTouchEnd={touchEnd} />;
}

```

#### 编译不区分平台

  `taro`的`dist`目录下不区分编译平台，同一时间仅可编译到一个平台，不支持多个平台对比查看运行效果,这个要注意

#### 案例都是微信小程序

  虽然`Taro`号称跨端平台，不过目前貌似使用者都是微信小程序，案例也是清一色的微信小程序，很少的H5和App的案例

### uni-app踩坑指南

因为物业管理部分需要使用[华创](http://homecommunity.cn/)，同时他们使用的是uni-app开发，为了能够合并华创的物业管理功能，因此决定将我们的项目使用`uni-app`重构，，代码仓库地址：[智慧社区](https://codeup.aliyun.com/5ff282e3f6558cb57ed8af06/ljgdwl/mcp)

这里记录一下使用`uni-app`过程中遇到的问题：

#### 使用HbuilderX唤起微信开发者工具失败

这个不算问题，但是需要注意，微信开发者工具默认是不支持其它工具唤起的，需要在`工具` -> `设置` -> `安全`设置中，将`服务端口`开启

#### 使用HbuilderX如何设置反向代理

修改根目录下的`manifest.json`，使用`源码试图`，修改`h5`的配置：

```json
{
  "h5": {
    "devServer": {
      "port": 80,
      "proxy": {
        "/api": {
          "target": "http://127.0.0.1:8080/xxx-services",
          "changeOrigin": true,
          "secure": false,
          "pathRewrite": {
            "^/api": ""
          }
        }
      }
    }
  }
}
```

#### 运行h5，控制台报错

这个问题是由于https和域名检测导致的，需要修改根目录下的`manifest.json`，使用`源码试图`，修改`h5`的配置：


```json
{
  "h5": {
    "devServer": {
      "port": 80,
      "disableHostCheck" : true,
      "https" : false
    }
  }
}
```

#### v-show打包不起作用

这个属于`vue`的属性，但是在`uni-app`打包之后无效，所以避免使用，使用`v-if`代替