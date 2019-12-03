---
title: 使用koa实现上传文件功能
date: 2019-12-01 21:41:15
categories: server
tags: koa
---

### 前言

文件上传，是写后端服务必须要遇到的一道坎儿，周末做chrome日志分析的时候，用到了上传功能，因为以前都是用java来写后端，这次改用nodejs写后端，遇到了一点小坎坷，特此做一个总结。

### koa介绍

Koa是基于Node.js的下一代web框架，由Express团队打造，特点：优雅、简洁、灵活、体积小。几乎所有功能都需要通过中间件实现。

1. Express是第一代最流行的web框架，它对Node.js的http进行了封装，用起来如下：

```javascript
var express = require('express');
var app = express();

app.get('/', function (req, res) {
    res.send('Hello World!');
});

app.listen(3000, function () {
    console.log('Example app listening on port 3000!');
});
```

虽然Express的API很简单，但是它是基于ES5的语法，要实现异步代码，只有一个方法：回调。如果异步嵌套层次过多，代码写起来就非常难看：

```javascript
app.get('/test', function (req, res) {
    fs.readFile('/file1', function (err, data) {
        if (err) {
            res.status(500).send('read file1 error');
        }
        fs.readFile('/file2', function (err, data) {
            if (err) {
                res.status(500).send('read file2 error');
            }
            res.type('text/plain');
            res.send(data);
        });
    });
});
```

2. koa 1.0

随着新版Node.js开始支持ES6，Express的团队又基于ES6的generator重新编写了下一代web框架koa。和Express相比，koa 1.0使用generator实现异步，代码看起来像同步的：

```javascript
var koa = require('koa');
var app = koa();

app.use('/test', function *() {
    yield doReadFile1();
    var data = yield doReadFile2();
    this.body = data;
});

app.listen(3000);
```

用generator实现异步比回调简单了不少，但是generator的本意并不是异步。Promise才是为异步设计的，但是Promise的写法……想想就复杂。为了简化异步代码，ES7引入了新的关键字`async`和`await`，可以轻松地把一个function变为异步模式：

```javascript
async function () {
    var data = await fs.read('/file1');
}
```

这是JavaScript未来标准的异步代码，非常简洁，并且易于使用。

3. koa2

koa团队并没有止步于koa 1.0，他们非常超前地基于ES7开发了koa2，和koa 1相比，koa2完全使用Promise并配合async来实现异步。

koa2的代码看上去像这样：

```javascript
app.use(async (ctx, next) => {
    await next();
    var data = await doReadFile();
    ctx.response.type = 'text/plain';
    ctx.response.body = data;
});
```

ES7是大势所趋，所以本次上传功能，直接使用基于ES7的koa2来实现。

### Stream流介绍

Stream 是一个抽象接口，Node 中有很多对象实现了这个接口。例如，对http 服务器发起请求的request 对象就是一个 Stream，还有stdout（标准输出）。

Stream 有四种流类型：

- Readable - 可读操作。
- Writable - 可写操作。
- Duplex - 可读可写操作.
- Transform - 操作被写入数据，然后读出结果。

所有的 Stream 对象都是 EventEmitter 的实例。常用的事件有：

- data - 当有数据可读时触发。
- end - 没有更多的数据可读时触发。
- error - 在接收和写入过程中发生错误时触发。
- finish - 所有数据已被写入到底层系统时触发。

上传的本质就是，客户端输入流，服务器端接收后输出流，我们上传要用的就是流中的管道流：用于从一个流中获取数据并将数据传递到另外一个流中。

```javascript
var fs = require("fs");

// 创建一个可读流
var readerStream = fs.createReadStream('input.txt');
// 创建一个可写流
var writerStream = fs.createWriteStream('output.txt');
// 管道读写操作
// 读取 input.txt 文件内容，并将内容写入到 output.txt 文件中
readerStream.pipe(writerStream);

console.log("程序执行完毕");
```

### 上传功能实现

```javascript
const fs = require('fs')
const path = require('path')
const Koa = require('koa')
const router = require('koa-router')()
const koaBody = require('koa-body'); //解析上传文件的插件

// 获取koa实例
const app = new Koa()

// 增加日志
app.use(async (ctx, next) => {
  console.log(`Process ${ctx.request.method} ${ctx.request.url}...`)
  await next()
})

// 设置上传
app.use(koaBody({
  multipart: true,
  formidable: {
    maxFileSize: maxSize * 1000 * 1024 * 1024    // 设置上传文件大小最大限制，默认10M
  }
}))

// 实现上传功能
router.post('/upload', async (ctx, next) => {
  const file = ctx.request.files.file; // 上传的文件在ctx.request.files.file

  // 修改文件的名称
  var myDate = new Date();
  var newFilename = myDate.getTime() + '-' + file.name;
  var targetPath = path.join(__dirname, '/upload/' + `${newFilename}`);
  // 创建可读流
  const reader = fs.createReadStream(file.path);
  //创建可写流
  const upStream = fs.createWriteStream(targetPath);
  // 可读流通过管道写入可写流
  reader.pipe(upStream);

  return ctx.body = { code: 200, data: { url: 'http://' + ctx.headers.host + '/' + newFilename, local: targetPath } };
});

// 添加路由配置
app.use(router.routes())
// 启动监听端口
app.listen(port)
console.log(`应用程序已经启动，访问地址:http://127.0.0.1:${port}`)

```

上面的实现，是一个异步的，即如果上传完成之后，立刻对该上传的文件做操作，文件本身是没有写入完成的，会导致程序异常，笔者就在这里被小坑了一把（也有可能是学艺不精导致的-_-||），所以如果需要确保上传文件肯定可用，需要将上传改为异步的，改动方法如下：

```javascript
// 实现上传功能
router.post('/upload', async (ctx, next) => {
  const file = ctx.request.files.file; // 上传的文件在ctx.request.files.file
  // 修改文件的名称
  var myDate = new Date();
  var newFilename = myDate.getTime() + '-' + file.name;
  var targetPath = path.join(__dirname, '/upload/' + `${newFilename}`);
  // 写入本身是异步的，这里改为同步方法，防止接下来的执行报错
  var writeFile = function() {
    return new Promise(function (resolve, reject) {
      // 创建可读流
      const reader = fs.createReadStream(file.path);
      //创建可写流
      const upStream = fs.createWriteStream(targetPath);
      // 可读流通过管道写入可写流
      reader.pipe(upStream);
      upStream.on('finish', () => {
        resolve('finish');
      });
    });
  }
  await writeFile();
  return ctx.body = { code: 200, data: { url: 'http://' + ctx.headers.host + '/' + newFilename, local: targetPath } };
});
```