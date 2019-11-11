---
title: vue版的抽奖大转盘
date: 2019-11-10 20:12:22
categories: server
tags: vuejs
---

### 前言
公司项目中要实现一个电视端的抽奖大转盘，本来想找现成的转盘拿来改动改动，结果发现存在两个问题：

1. 大部分抽奖转盘的demo均为采用jquery实现，目前项目中用的是vue，为了一个转盘引入jquery，成本太大
2. 转盘的动画表现，采用了js的动画，目前项目中的电视机顶盒配置不高，采用js动画，卡顿现象比较严重

因此，决定参考其他demo，自己写一个，目前项目已经使用，特意把相关代码抽出来，做一个demo，以供大家参考

### 需求

1. 采用vue实现
2. 奖品数量不固定，要尽可能做到不固定数量的奖品的兼容（少到撑不起一个转盘或多到转盘每一列连字都放不下不再考虑范围）
3. js动画在低端的机顶盒上表现不好，为了性能，应该采用css动画来实现，不能用js实现
4. 中奖概率由后台控制抽取，而不是通过js随机

## 解决方案

1. 通过canvas来画抽奖大转盘，可以通过数量计算每个奖品的角度
2. 通过transition来实现抽奖转盘的动画，采用过渡效果为：`transform 8s ease-in-out`，慢-快-慢，来模拟抽奖转盘

## 效果预览

![](/images/other/lottery.gif)

## 代码部分

#### 背景灯

> 这里通过setTimeout，来让两张背景图切换，来实现跑马灯的效果（如果有更好的css动画能实现这个效果，欢迎大家在[github](https://github.com/yxl2628/blog/issues)上给我留言）

```html
<div class="bg">
  <img v-show="bg" class="bg1" src="./image/bg1.png" alt="" />
  <img v-show="!bg" class="bg2" src="./image/bg2.png" alt="" />
</div>
```

```javascript
new Vue({
  el: '#root',
  data() {
    bg: true,
  },
  mounted() {
    setInterval(() => {
      this.bg = !this.bg;
    }, 1000);
  }
})
```

#### canvas转盘
```html
<div class="wheel-bg">
  <canvas id="wheelCanvas" width="420px" height="420px" class="lottery-bg"
    :style="`transform: rotate(${wheelDeg}deg)`"></canvas>
</div>
```

```css
.body .wheel-bg {
  position: absolute;
  top: 50%;
  left: 50%;
  width: calc(100% - 60px);
  height: calc(100% - 60px);
  background-color: #ab3212;
  border-radius: 50%;
  overflow: hidden;
  transform: translate(-50%, -50%);
  transition: transform 8s ease-in-out;
  z-index: 1;
}

.body .lottery-bg {
  position: absolute;
  top: 10px;
  left: 10px;
  width: calc(100% - 20px);
  height: calc(100% - 20px);
  background-color: #fff;
  border-radius: 50%;
  overflow: hidden;
  transition: transform 8s ease-in-out;
  z-index: 2;
}
```

```javascript
//获取canvas画布
// this.list 是通过后台获取回来的奖品列表
var len = this.list.length;
var canvas = document.getElementById("wheelCanvas");
var ctx = canvas.getContext("2d");
var canvasW = canvas.width; // 画板的高度
var canvasH = canvas.height; // 画板的宽度
//计算每个奖项所占角度数
var baseAngle = Math.PI * 2 / len;
ctx.clearRect(0, 0, canvasW, canvasH); //去掉背景默认的黑色
ctx.strokeStyle = "#ffb725"; //设置画图线的颜色
// 开始循环绘制每个奖品
for (var i = 0; i < len; i++) {
// 每个扇形的起始角度
var angle = i * baseAngle;
ctx.font = '16px Microsoft YaHei'; //设置字号字体
ctx.fillStyle = i % 2 === 0 ? '#ffdb37' : '#ffb725'; //设置每个扇形区域的颜色
ctx.beginPath(); //开始绘制
ctx.arc(canvasW * 0.5, canvasH * 0.5, 200, angle, angle + baseAngle, false);
// 中心点的小圆，这样会形成一个环形图案
ctx.arc(canvasW * 0.5, canvasH * 0.5, 10, angle + baseAngle, angle, true);
ctx.stroke(); //开始链线
ctx.fill(); //填充颜色
ctx.save(); //保存当前环境的状态
ctx.fillStyle = "#e9311f"; // 设置文字颜色
var item = this.list[i];
var line_height = 24;
// 计算文字的偏移量
var translateX = canvasW * 0.5 + Math.cos(angle + baseAngle / 2) * 260;
var translateY = canvasH * 0.5 + Math.sin(angle + baseAngle / 2) * 260;
// 如果奖品无图，则文字相应改大，并且偏移变小
if (!item.img) {
  ctx.font = '20px Microsoft YaHei'; //设置字号字体
  translateX = canvasW * 0.5 + Math.cos(angle + baseAngle / 2) * 220;
  translateY = canvasH * 0.5 + Math.sin(angle + baseAngle / 2) * 220;
}
ctx.translate(translateX, translateY);
// rotate方法旋转当前的绘图，因为文字适合当前扇形中心线垂直的！
// angle，当前扇形自身旋转的角度 +  baseAngle / 2 中心线多旋转的角度  + 垂直的角度90°
ctx.rotate(angle + baseAngle / 2 + Math.PI / 2);
//设置文本位置，居中显示 
ctx.fillText(item.name, -ctx.measureText(item.name).width / 2, 100);
//添加对应缩略图
var triangleEdge = canvasH * 0.27;
// 根据角度不同，计算图片最大宽度
var imgMaxWidth = Math.sqrt(2 * triangleEdge * triangleEdge * (1 - Math.cos(baseAngle)));
if (item.imgEl) {
  ctx.drawImage(item.imgEl, -imgMaxWidth * 0.5, canvasH * 0.27, imgMaxWidth, item.img
    .vertical_resolution * imgMaxWidth / item.img.horizontal_resolution);
}
ctx.restore(); //很关键，还原画板的状态到上一个save()状态之前
```

#### 抽奖程序
```javascript
// 这里应该从后台获取中奖结果，我们随便模拟一下中奖结果（这里应该按照实际项目情况，做实际处理）
var winIndex = parseInt(Math.random() * (this.list.length + 1), 10);
this.result = this.list[winIndex];
if (this.result.name === '谢谢参与') {
  this.result = null;
}
// 计算中奖的旋转角度
this.wheelDeg = this.wheelDeg - this.wheelDeg % 360 + 3600 + (360 - 360 / this.list.length * winIndex) - (
  180 - 360 / this.list.length);
// 显示中奖结果(因为我设定的转盘动画是8s，所以显示中奖结果的时间设定为8.5s)
setTimeout(() => {
  this.showResult = true;
  this.rolling = false;
}, 8500)
```

### 相关公式

计算图片最大宽度用了正余弦定理，通过两边长度和角度，计算第三边长度

[](/images/other/cosA.png)