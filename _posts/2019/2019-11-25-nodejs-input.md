---
title: nodejs获取命令行输入
date: 2019-11-25 10:02:53
categories: javascript
tags: nodejs
---

### readline介绍

官网链接：[http://nodejs.cn/api/readline#readline_readline](http://nodejs.cn/api/readline#readline_readline)

require('readline') 模块提供了一个接口，用于从可读流（如 process.stdin）读取数据，每次读取一行。 它可以通过以下方式使用：
`const readline = require('readline');`

例子，readline 模块的基本用法：

```javascript
const readline = require('readline');

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

rl.question('你认为 Node.js 中文网怎么样？', (answer) => {
  // 对答案进行处理
  console.log(`多谢你的反馈：${answer}`);

  rl.close();
});
```

### 实战：比大小游戏

代码如下：

```javascript
const readline = require('readline');

readSyncByRl('please input your name：').then((res) => {
  console.log(`Hello, ${res}!`);
  console.log(`I'll give you a number, you guess it, now, let's begin:`);
  const win = parseInt(Math.random() * 999, 10) + 1;
  guess(win);
});


/**
 * 比大小游戏
 *
 * @author yang.xiaolong
 * @date 2019-11-25
 * @param {*} win
 */
function guess(win) {
  readSyncByRl('please input your answer：').then((res) => {
    if (res < win) {
      console.log(`small. Please guess a little bigger number.`);
      guess(win);
    } else if (res > win) {
      console.log(`big. Please guess a little smaller number.`);
      guess(win);
    } else {
      console.log(`Congratulations. You Win.`);
    }
  });
}

/**
 * 封装读取命令行输入
 *
 * @author yang.xiaolong
 * @date 2019-11-25
 * @param {*} tips
 * @returns
 */
function readSyncByRl(tips) {
  tips = tips || '> ';
  return new Promise((resolve) => {
    const rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    });

    rl.question(tips, (answer) => {
      rl.close();
      resolve(answer.trim());
    });
  });
}
```