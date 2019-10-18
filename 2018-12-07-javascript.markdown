---
layout: post
title: js相关的小技巧及简单api
date: 2018-12-7 13:47:39
tags: javascript
---

### 判断内外网

```javascript
var baseUrl = (function(){
  var privateReg = /^1(((0|27)(.(([1-9]?|1[0-9])[0-9]|2([0-4][0-9]|5[0-5])))|(72.(1[6-9]|2[0-9]|3[01])|92.168))(.(([1-9]?|1[0-9])[0-9]|2([0-4][0-9]|5[0-5]))){2})$/
  var private = 'http://内网ip'
  var internet = 'http://外网ip'
  return privateReg.test(window.location.hostname) ? private : internet
})()
```

### js获取url参数（含hash）

```javascript
function GetQueryString(param) {
  const reg = new RegExp('(^|&)' + param + '=([^&]*)(&|$)');
  const r = window.location.search.substr(1).match(reg) || window.location.hash.substring((window.location.hash.search(/\?/)) + 1).match(reg);
  if (r != null) {
    return decodeURIComponent(r[2]);
  }
}
```

### 当前浏览器兼容Promise方案

  1. 可以引入`<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-polyfill/7.2.5/polyfill.js"></script>`,但是缺点是：无论浏览器是否支持，都全量引入了，对已经支持Promise的用户来说，是一种带宽和流量的浪费
  2. 追求极致性能的话，可以选择按需引入<https://github.com/zloirock/core-js/tree/v2/modules>下的文件
  3. 在线补丁：最流行的：<https://polyfill.io/v3/>,引入方式为：`<script src="https://polyfill.io/v3/polyfill.min.js?features=default%2CPromise"></script>` 以及阿里CBU团队基于此搭建的alicdn：`<script src="https://polyfill.alicdn.com/polyfill.min.js?features=Promise"></script>`

### js生成随机数

  > 来自 http://www.cnblogs.com/starof/p/4988516.html

  1. 生成[1,max]的随机数，公式如下：
  ```javascript
  // max - 期望的最大值
  parseInt(Math.random()*max,10)+1;
  Math.floor(Math.random()*max)+1;
  Math.ceil(Math.random()*max);
  ```
  2. 生成[0,max]到任意数的随机数，公式如下：
  ```javascript
  // max - 期望的最大值
  parseInt(Math.random()*(max+1),10);
  Math.floor(Math.random()*(max+1));
  ```
  3. 生成[min,max]的随机数，公式如下：
  ```javascript
  // max - 期望的最大值
  // min - 期望的最小值
  parseInt(Math.random()*(max-min+1)+min,10);
  Math.floor(Math.random()*(max-min+1)+min);
  ```