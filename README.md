# Node.js

## 介绍

基于Google V8引擎的JavaScript运行时环境，同时结合Libuv扩展了JavaScript功能，使之成为io、fs等只有语言才有的特性。同时拥有DOM操作和I/O、文件读写、操作数据库（服务器端）等能力。

### 特点：

事件驱动、非阻塞IO模型（异步）、轻量和高效。

<img src="README.assets/image-20210610160234052.png" alt="image-20210610160234052" style="zoom: 33%;" />

### 使用：

1. 做中间层
   1). 减轻客户端内存，不会像mvvm模式的项目把页面渲染和数据请求都压在客户端，而是在服务端完成。
   2). SEO性能好，服务端渲染好html字符，有利于网页被搜索到。
   3). 前端可操控范围增多，甚至可以做服务器，

2. 做项目构建工具
   webpack、vue-cli都是输入项目构建工具，通过node来写的。

3. 做小型网站后端

## 模块化

### 模块化的好处

​	低耦合、高内聚：提高重复利用率
​	方便维护：模块化便于管理
​	防止代码重复：通过必报的形式来保护变量不收外界干扰。

### CommonJS

Node.js基于CommonJS实现原理是文件的读写，使用exports、require、module

#### require

```js
// require node和es6都支持的引入
let a = require('./test.js')

//第三方包名则优先在同级目录的node_modules下查找第三方包
let template = require('art-template')
// 通过第三方包中的package.json文件找到里面的main属性对应的入口模块，该入口模块即为加载的第三方模块
//若没有找到node_modules文件夹，或者上述情况都没有找到，则会向上一级目录查找node_modules
//如果到该模块根路径都没有找到，则会报错：
//can not find module xxx
```

#### exports和module.exports

在node执行文件时，会给文件生成exports和module对象，而module又有一个exports属性，都指向同一块内存区域。

<img src="README.assets/view.png" alt="preview" style="zoom: 50%;" />

```js
// exports = module.exports = {};
// exports只有es6支持的导出 module.exports只有node支持的导出
exports.test = function() {
  console.log("test")
}

//utils.js
let a = 100;
console.log(module.exports); //能打印出结果为：{}
console.log(exports); //能打印出结果为：{}
exports.a = 200; //这里辛苦劳作帮 module.exports 的内容给改成 {a : 200}
exports = '指向其他内存区'; //这里把exports的指向指走
//test.js
var a = require('/utils');
console.log(a) // 打印为 {a : 200}
```

#### export和export default

1. export和export default均可用于导出常量、函数、文件、模块等。
2. export、import可以有多个，export default**只能有一个**
3. 通过export方式导出，**导入时要加 { }** ，export default不需要
4. export可以**直接导出变量表达式**，export default不行。

```js
//testEs6Export.js
//导出变量
export const a = '100';
//导出函数方式1
export const test1 = function() {
  //....
}
//导出函数方式2
function test2() {
  //....
};
export { test2 }; //或者也可以使用exports.test2 = test2
// 不可以使用export default const b = 100
const b = 100;
export default b;

//main.js
import { test, test2 } from './testEs6Export' //导出export
import b from './testEs6Export' //导出export default
//通过as集合导出对象
import * as testModule from './testEs6Export'
//! 注意default是导出为default属性，as仅是通过聚合export成一个对象。
testModule.b //undefined
testModule.default //100
```

### NPM

npm：node package manager

```js
//基本操作
npm init //初始化package.json文件
npm install //下载package.json中所有依赖
npm install xxx --save-dev //仅用于开发环境，出现在devDependencies属性中
npm isntall xxx --save //用于生产环境，出现在dependencies属性中
npm list //已安装的node包
npm info xxx //查看远程npm上指定包的所有版本信息
npm root //查看当前包的安装路径
npm root -g //查看全局的包的安装路径
npm ls xx //查看本地包及版本信息
npm ls xx -g
```

## FS 文件系统

### 读取文件

```js
const fs= require('fs')

//同步读取
var data = fs.readFileSync('hello.txt');
//异步读取
fs.readFile('hello.txt',function(err, data){
  if(err) throw err;
  console.log(data);
})
fs.writeFile('test.txt','helloworld',{flag:'a', encoding:'utf-8'}, function(err){
  if(err) console.log("error",err)
})
```

### 打开文件

```js
// fs.open(path, flags[, mode], callback)
fs.open('input.txt', 'r+', function(err, fd) {
  if(err) {
    return console.error(err);
  }
  console.log("打开成功！")
});
```

- **path** - 文件的路径。
- **flags** - 文件打开的行为。具体值详见下文。
- **mode** - 设置文件模式(权限)，文件创建默认权限为 0666(可读，可写)。
- **callback** - 回调函数，带有两个参数如：callback(err, fd)。

| Flag | 描述                                                   |
| ---- | ------------------------------------------------------ |
| r    | 以读取模式打开文件。如果文件不存在抛出异常。           |
| r+   | 以读写模式打开文件。如果文件不存在抛出异常。           |
| rs   | 以同步的方式读取文件。                                 |
| rs+  | 以同步的方式读取和写入文件。                           |
| w    | 以写入模式打开文件，如果文件不存在则创建。             |
| wx   | 类似 'w'，但是如果文件路径存在，则文件写入失败。       |
| w+   | 以读写模式打开文件，如果文件不存在则创建。             |
| wx+  | 类似 'w+'， 但是如果文件路径存在，则文件读写失败。     |
| a    | 以追加模式打开文件，如果文件不存在则创建。             |
| ax   | 类似 'a'， 但是如果文件路径存在，则文件追加失败。      |
| a+   | 以读取追加模式打开文件，如果文件不存在则创建。         |
| ax+  | 类似 'a+'， 但是如果文件路径存在，则文件读取追加失败。 |



### 获取文件信息

```js
// fs.stat(path, callback)
fs.stat('test.js', function(err, stats) {
  console.log(stats.isFile());
})
```

states类中方法有：

| 方法                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| stats.isFile()            | 如果是文件返回 true，否则返回 false。                        |
| stats.isDirectory()       | 如果是目录返回 true，否则返回 false。                        |
| stats.isBlockDevice()     | 如果是块设备返回 true，否则返回 false。                      |
| stats.isCharacterDevice() | 如果是字符设备返回 true，否则返回 false。                    |
| stats.isSymbolicLink()    | 如果是软链接返回 true，否则返回 false。                      |
| stats.isFIFO()            | 如果是FIFO，返回true，否则返回 false。FIFO是UNIX中的一种特殊类型的命令管道。 |
| stats.isSocket()          | 如果是 Socket 返回 true，否则返回 false。                    |

### 写入文件

```js
// fs.writeFile(file, data[, options], callback)
fs.writeFile('input.txt', '我是通过fs.writeFile 写入文件的内容',  function(err) {
   if (err) {
       return console.error(err);
   }
   fs.readFile('input.txt', function (err, data) {
      if (err) {
         return console.error(err);
      }
      console.log("异步读取文件数据: " + data.toString());
   });
});
```

- **file** - 文件名或文件描述符。
- **data** - 要写入文件的数据，可以是 String(字符串) 或 Buffer(缓冲) 对象。
- **options** - 该参数是一个对象，包含 {encoding, mode, flag}。默认编码为 utf8, 模式为 0666 ， flag 为 'w'
- **callback** - 回调函数，回调函数只包含错误信息参数(err)，在写入失败时返回。

### 读取文件

```js
// fs.read(fd, buffer, offset, length, position, callback)
```

- **fd** - 通过 fs.open() 方法返回的文件描述符。
- **buffer** - 数据写入的缓冲区。
- **offset** - 缓冲区写入的写入偏移量。
- **length** - 要从文件中读取的字节数。
- **position** - 文件读取的起始位置，如果 position 的值为 null，则会从当前文件指针的位置读取。
- **callback** - 回调函数，有三个参数err, bytesRead, buffer，err 为错误信息， bytesRead 表示读取的字节数，buffer 为缓冲区对象。

### 关闭文件

```js
// fs.close(fd, callback)
var fs = require("fs");
var buf = new Buffer.alloc(1024);
fs.open('input.txt', 'r+', function(err, fd) {
   if (err) {
       return console.error(err);
   }
   fs.read(fd, buf, 0, buf.length, 0, function(err, bytes){
      if (err){
         console.log(err);
      }
      // 仅输出读取的字节
      if(bytes > 0){
         console.log(buf.slice(0, bytes).toString());
      }
      // 关闭文件
      fs.close(fd, function(err){
         if (err){
            console.log(err);
         } 
         console.log("文件关闭成功");
      });
   });
});
```

- **fd** - 通过 fs.open() 方法返回的文件描述符。
- **callback** - 回调函数，没有参数。

### 截取文件

```js
// fs.ftruncate(fd, len, callback)
fs.ftruncate(fd, 10, function(err){
  if (err){
    console.log(err);
  } 
});
```

- **fd** - 通过 fs.open() 方法返回的文件描述符。
- **len** - 文件内容截取的长度。
- **callback** - 回调函数，没有参数。

### 删除文件

```js
// fs.unlink(path, callback)
fs.unlink('input.txt', function(err) {
   if (err) {
       return console.error(err);
   }
   console.log("文件删除成功！");
});
```

### 创建目录

```js
// fs.mkdir(path[, options], callback)
fs.mkdir("/tmp/test/",function(err){
   if (err) {
       return console.error(err);
   }
   console.log("目录创建成功。");
});
```

- **path** - 文件路径。
- options 参数可以是：
  - **recursive** - 是否以递归的方式创建目录，默认为 false。
  - **mode** - 设置目录权限，默认为 0777。
- **callback** - 回调函数，没有参数。

### 读取目录

```js
// fs.readdir(path, callback)
fs.readdir("/tmp/",function(err, files){
   if (err) {
       return console.error(err);
   }
   files.forEach( function (file){
       console.log( file );
   });
});
```

- **path** - 文件路径。
- **callback** - 回调函数，回调函数带有两个参数err, files，err 为错误信息，files 为 目录下的文件数组列表。

### 删除目录

```js
// fs.rmdir(path, callback)
fs.rmdir("/tmp/test",function(err){
   if (err) {
       return console.error(err);
   }
});
```

## Stream 流

Node.js有四种流类型：

- Readable - 可读操作
- Writeable - 可写操作
- Duplex - 可读可写操作
- Transform - 操作被写入数据，然后读出结果。

所有的Stream对象都是EventEmitter的实例，常用的事件有：

- data：当有数据可读时触发
- end：没有更多的数据可读时触发
- error：在接收和写入过程中发生错误时触发
- finish：所有数据已被写入到底层系统时触发

### 从流中读取数据

```js
var fs = require("fs");
var data = '';
// 创建可读流
var readerStream = fs.createReadStream('input.txt');
// 设置编码为 utf8。
readerStream.setEncoding('UTF8');
// 处理流事件 --> data, end, and error
readerStream.on('data', function(chunk) {
   data += chunk;
});
readerStream.on('end',function(){
   console.log(data);
});
readerStream.on('error', function(err){
   console.log(err.stack);
});
console.log("程序执行完毕");
```

### 写入流

```js
var fs = require("fs");
var data = 'hello world this is test';
// 创建一个可以写入的流，写入到文件 output.txt 中
var writerStream = fs.createWriteStream('output.txt');
// 使用 utf8 编码写入数据
writerStream.write(data,'UTF8');
// 标记文件末尾
writerStream.end();
// 处理流事件 --> finish、error
writerStream.on('finish', function() {
    console.log("写入完成。");
});
writerStream.on('error', function(err){
   console.log(err.stack);
});
console.log("程序执行完毕");
```

### 管道流

管道提供了一个输出流到输入流的机制。通常我们用于从一个流中获取数据并将数据传递到另外一个流中。

```js
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

#### 链式流

链式是通过连接输出流到另外一个流并创建多个流操作链的机制。链式流一般用于管道操作。

```js
//链式写入
var fs = require("fs");
var zlib = require('zlib');
// 压缩 input.txt 文件为 input.txt.gz
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));
console.log("文件压缩完成。");

//链式输出
var fs = require("fs");
var zlib = require('zlib');
// 解压 input.txt.gz 文件为 input.txt
fs.createReadStream('input.txt.gz')
  .pipe(zlib.createGunzip())
  .pipe(fs.createWriteStream('input.txt'));
console.log("文件解压完成。");
```

## Events 事件

Node.js几乎每一个API都支持回调函数，基本上都是用观察者模式实现的。

<img src="README.assets/event_loop-20210615085321996.jpg" alt="img" style="zoom: 67%;" />

### EventEmitters

通过引入events模块，并实例化EventEmitter类来绑定和监听事件。

```js
var events = require('events');
var eventEmitter = new events.EventEmitter();
// 绑定事件及处理程序
eventEmitter.on('eventName',eventHandler);
// 触发事件
eventEmitter.emit('eventName')
```

#### EventEmitter的属性

##### 方法

| 序号 | 方法 & 描述                                                  |
| ---- | ------------------------------------------------------------ |
| 1    | **addListener(event, listener)** 为指定事件添加一个监听器到监听器数组的尾部。 |
| 2    | **on(event, listener)** 为指定事件注册一个监听器，接受一个字符串 event 和一个回调函数。 |
| 3    | **once(event, listener)** 为指定事件注册一个单次监听器，即 监听器最多只会触发一次，触发后立刻解除该监听器。 |
| 4    | **removeListener(event, listener)** 移除指定事件的某个监听器，监听器必须是该事件已经注册过的监听器。它接受两个参数，第一个是事件名称，第二个是回调函数名称。 |
| 5    | **removeAllListeners([event])** 移除所有事件的所有监听器， 如果指定事件，则移除指定事件的所有监听器。 |
| 6    | **setMaxListeners(n)** 默认情况下， EventEmitters 如果你添加的监听器超过 10 个就会输出警告信息。 setMaxListeners 函数用于改变监听器的默认限制的数量。 |
| 7    | **listeners(event)** 返回指定事件的监听器数组。              |
| 8    | **emit(event, [arg1], [arg2], [...])** 按监听器的顺序执行执行每个监听器，如果事件有注册监听返回 true，否则返回 false。 |

##### 类方法

| 序号 | 方法 & 描述                                                  |
| ---- | ------------------------------------------------------------ |
| 1    | **listenerCount(emitter, event)** 返回指定事件的监听器数量。 |

##### 事件

| 序号 | 事件 & 描述                                                  |
| ---- | ------------------------------------------------------------ |
| 1    | **newListener**<br />**event** - 字符串，事件名称<br />**listener** - 处理事件函数该事件在添加新监听器时被触发。 |
| 2    | **removeListener** <br />**event** - 字符串，事件名称<br />**listener** - 处理事件函数从指定监听器数组中删除一个监听器。需要注意的是，此操作将会改变处于被删监听器之后的那些监听器的索引。 |

#### 继承 EventEmitter

大多数时候我们不会直接使用 EventEmitter，而是在对象中继承它。包括 fs、net、 http 在内的，只要是支持事件响应的核心模块都是 EventEmitter 的子类。
原因有两点：
首先，具有某个实体功能的对象实现事件符合语义， 事件的监听和发生应该是一个对象的方法。
其次 JavaScript 的对象机制是基于原型的，支持 部分多重继承，继承 EventEmitter 不会打乱对象原有的继承关系。

## 工具模块

### Path模块

```js
let path = require('path');

// 获取后缀名，即最后一个'.'之后的部分
let strPath = "http://www.test.com/a.jpg";
let info = path.extname( strPath ); //.jpg

//解析为绝对路径
path.resolve('/nodejs','study','modules'); // C:\nodejs\study\modules

//规范化路径
path.normalize(p)

//判断是否是绝对路径
path.isAbsolute(p)

//绝对路径转化为相对路径，从from到to path.relative(from,to)
path.relative('data/test/aaa','data/impl/bbb') // 返回 '../../impl/bbb'

//返回路径中代表文件夹的部分
path.dirname(p)

//返回路径中的最后一部分
path.basename(p[,ext])

//返回路径字符串对象
path.parse(pathString)

//从对象中返回路径字符串，和path.parse相反
path.format(pathObject)

//获取路径和脚本名
console.log(__filename) //执行文件的文件路径，为绝对路径
console.log(__dirname) //当前执行脚本所在的目录
process.cwd() //当前执行node命令时候的文件夹目录名
```

### OS模块

| 序号 | 方法 & 描述                                                  |
| ---- | ------------------------------------------------------------ |
| 1    | **os.tmpdir()** 返回操作系统的默认临时文件夹。               |
| 2    | **os.endianness()** 返回 CPU 的字节序，可能的是 "BE" 或 "LE"。 |
| 3    | **os.hostname()** 返回操作系统的主机名。                     |
| 4    | **os.type()** 返回操作系统名                                 |
| 5    | **os.platform()** 返回编译时的操作系统名                     |
| 6    | **os.arch()** 返回操作系统 CPU 架构，可能的值有 "x64"、"arm" 和 "ia32"。 |
| 7    | **os.release()** 返回操作系统的发行版本。                    |
| 8    | **os.uptime()** 返回操作系统运行的时间，以秒为单位。         |
| 9    | **os.loadavg()** 返回一个包含 1、5、15 分钟平均负载的数组。  |
| 10   | **os.totalmem()** 返回系统内存总量，单位为字节。             |
| 11   | **os.freemem()** 返回操作系统空闲内存量，单位是字节。        |
| 12   | **os.cpus()** 返回一个对象数组，包含所安装的每个 CPU/内核的信息：型号、速度（单位 MHz）、时间（一个包含 user、nice、sys、idle 和 irq 所使用 CPU/内核毫秒数的对象）。 |
| 13   | **os.networkInterfaces()** 获得网络接口列表。                |

## 爬虫 cheerio

```js
const cheerio = require('cheerio');
const axios = require('axios');
const fs = require('fs');
const path = require('path');

let httpUrls = "https://www.doutula.com/article/list/?page="

async function getNum(){
  res = await axios.get(httpUrls);
  let $ = cheerio.load(res.data);
  let btnLength = $('.pagination li').length;
  let allNum = $('.pagination li').eq(btnLength-2).find('a').text()
  return allNum
}

async function spider(){
  let allPageNum = await getNum();
  for(let i=1;i<=allPageNum;i++){
    getListPage(i);
  }
}

async function getListPage(pageNum){
  let httpUrl = httpUrls + pageNum;
  let res = await axios.get(httpUrl);
  let $ = cheerio.load(res.data)
  $('#home .col-sm-9 a').each((i,element)=>{
    let pageUrl = $(element).attr('href');
    let title = $(element).find('.random_title').text()
    let reg = /(.*?)\d/igs;
    title = reg.exec(title)[1];
    fs.mkdir('./img/'+title,function(err){
      if(err) console.log(err);
    });
    parsePage(pageUrl);
  })
}

async function parsePage(url,title){
  let res = await axios.get(url);
  let $ = cheerio.load(res.data)
  $('.pic-content img').each((i,element)=>{
    let imgUrl = $(element.attr('src'))
    let extName = path.extname(imgUrl);
    let ws = fs.createWriteStream(`./img/${title}/${title}-${i}${extName}`)
    axios.get(imgUrl,{responseType:'stream'}).then(function(){
      res.data.pipe(ws);
      res.data.on('close',function(){
        ws.close();
      })
    })
  })
}

spider();
```



