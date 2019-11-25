---
title: pta上，nodejs考核遇到的坑
date: 2019-11-25 11:33:03
categories: javascript
tags: nodejs
---

### PTA介绍

PTA全名：程序设计类实验辅助教学平台，网址：[https://pintia.cn](https://pintia.cn) 是一个提供在线程序试题测评的教学平台。

> 恰逢公司技术水平考核，需要在pta上进行考核，前端嘛，要使用nodejs来考核，偏偏nodejs不像java、python，System.in/input() 就能拿到命令行入参，nodejs显然要费事一点，所以写一篇blog，以免后来者入坑。

### PTA中提交Nodejs程序的一些套路

node.js是使用`process.stdin`来进行命令行输入输出获取的，但是获取的参数，要自己来进行处理：

1. 引入`fs`

```javascript
var fs = require('fs');
```

2. 定义输入字符串变量

```javascript
var buf = '';
```

3. 监听输入流

```javascript
process.stdin.on('readable', function () {
  var chunk = process.stdin.read();
  if (chunk) buf += chunk.toString();
});
```

4. 处理输入

```javascript
process.stdin.on('end', function () {
  // 通过回车符来分隔每一行输入
  buf.split('\n').forEach(function (line) {
    // 通过空格来分隔单行输入中的多个参数，如果明确是一个入参，则可以直接使用：var param = parseInt(line);
    var params = line.split(' ').map(function (x) { return parseInt(x); });
    // 这里用来排除错误输入，不加这行，至少我做的那道题百分百报错
    if (params.length != 2) return;
    // 将入参传入待测试方法
    console.log(testFn(params));
  });
});
```

5. 实例

用pta上这道题来做测试：[计算工资](https://pintia.cn/problem-sets/14/problems/790)

代码如下：

```javascript
var fs = require('fs');
var buf = '';

process.stdin.on('readable', function () {
  var chunk = process.stdin.read();
  if (chunk) buf += chunk.toString();
});

process.stdin.on('end', function () {
  buf.split('\n').forEach(function (line) {
    var arr = line.split(' ').map(function (x) { return parseInt(x); });
    if (arr.length != 2) return;
    console.log(testFn(arr));
  });
});


/**
 *  主要函数写在这里即可
 * 
 *  题目：某公司员工的工资计算方法如下：
 *    一周内工作时间不超过40小时，按正常工作时间计酬；
 *    超出40小时的工作时间部分，按正常工作时间报酬的1.5倍计酬。
 *    员工按进公司时间分为新职工和老职工，进公司不少于5年的员工为老职工，5年以下的为新职工。
 *    新职工的正常工资为30元/小时，老职工的正常工资为50元/小时。请按该计酬方式计算员工的工资。
 *
 *  输入：输入在一行中给出2个正整数，分别为某员工入职年数和周工作时间，其间以空格分隔。
 *  输出：在一行输出该员工的周薪，精确到小数点后2位。
 * 
 * @author yang.xiaolong
 * @date 2019-11-25
 * @param {*} year
 * @param {*} hour
 * @returns
 */
function testFn(arr) {
  const [year, hour] = arr;
  // 基本工资
  const baseMoney = year < 5 ? 30 : 50;
  let sum = 0;
  // 工作不超时
  if (hour <= 40) {
    sum = baseMoney * hour;
  } else { // 工作超时
    sum = baseMoney * 40 + baseMoney * 1.5 * (hour - 40);
  }
  // 返回结果，并保留小数点2位
  return sum.toFixed(2);
}
```