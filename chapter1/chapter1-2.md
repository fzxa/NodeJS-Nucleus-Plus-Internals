### Node Stream模块分析


Stream在平时业务开发时很少用到， 但是很多模块都是基于stream实现的，引用官方文档的解释：

流（stream）在 Node.js 中是处理流数据的抽象接口（abstract interface）。

stream 模块提供了基础的 API 。使用这些 API 可以很容易地来构建实现流接口的对象。

流可以是可读的、可写的，或是可读写的。所有的流都是 EventEmitter 的实例。



#### 为什么应该使用Stream

先来看一段代码:

这段代码有什么问题， 看似是没有问题的。

如果data.txt文件体积非常大，nodejs读入内存当中，然后全部取出

这样会对性能造成很大影响

```js
var http = require('http');
var fs = require('fs');

var server = http.createServer(function (req, res) {
    fs.readFile(__dirname + '/data.txt', function (err, data) {
        res.end(data);
    });
});
server.listen(8000);
```


经过优化后代码如下:

这段将data.txt一段一段的发送到用户端

这样减少了很多的服务器压力

```js
var http = require('http');
var fs = require('fs');

var server = http.createServer(function (req, res) {
    let stream = fs.createReadStream(__dirname + '/data.txt');//创造可读流
    stream.pipe(res);//将可读流写入response
});
server.listen(8000);
```


### 管道流Pipe
管道提供了一个输出流到输入流的机制, 从获取到数据传入另外一个流
无论哪一种流，都会使用.pipe()方法来实现输入和输出。

读取input.txt文件流 

hello World 

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
查看输出文件流output.txt

hello World



#### 如果多文件还可进行链式操作：

代码如下：
```js
a.pipe(b);
b.pipe(c);
c.pipe(d);

//上面代码等价于这样
a.pipe(b).pipe(c).pipe(d)
```

### readable 可读操作
Readable流可以产出数据流，你可以将这些数据传送到一个writable，transform， duplex，并调用pipe()方法:

```js
var Readable = require('stream').Readable;

var rs = new Readable;
rs.push('beep ');
rs.push('boop\n');
rs.push(null);

rs.pipe(process.stdout); //输出： beep boop
```
在上面的代码中rs.push(null)的作用是告诉rs输出数据应该结束了。


```js
var Readable = require('stream').Readable;
var rs = Readable();

var c = 97;
rs._read = function () {
    rs.push(String.fromCharCode(c++));
    if (c > 'z'.charCodeAt(0)) rs.push(null);
};

rs.pipe(process.stdout);//输出 abcdefghijklmnopqrstuvwxyz
```

还可以通过监听事件readable，触发时手工读取chunk数据:

一旦注册了readable事件，必须手工读取read数据，否则数据就会流失

```js
var Read = require('stream').Readable;
var r = new Read();

r.push('hello');
r.push('world');
r.push(null);

r.on('readable', function () {
    var chunk = r.read();
    console.log('get data by readable event: ', chunk.toString())
});

// get data by readable event:  hello world!
```
#### 注意：process.stdout之前已经将内容推送进readable流rs中，但是所有的数据依然是可写的

### Readable Stream的模式

Readable Stream 存在两种模式(flowing mode 与 paused mode)，
这两种模式决定了chunk数据流动的方式---自动流动还是手工流动。那如何触发这两种模式呢：

flowing mode: 注册事件data、调用resume方法、调用pipe方法

paused mode: 调用pause方法(没有pipe方法)、移除data事件 && unpipe所有pipe
```js
// data事件触发flowing mode
Readable.prototype.on = function(ev, fn) {
    ...
    if (ev === 'data' && false !== this._readableState.flowing) {
        this.resume();
      }
      ...
}

// resume触发flowing mode
Readable.prototype.resume = function() {
    var state = this._readableState;
    if (!state.flowing) {
           debug('resume');
           state.flowing = true;
    resume(this, state);
  }
  return this;
}

// pipe方法触发flowing模式
Readable.prototype.resume = function() {
    if (!state.flowing) {
        this.resume()
    }
}
```
结论

两种方式取决于一个flowing字段：true --> flowing mode；false --> paused mode

三种方式最后均是通过resume方法，将state.flowing = true

```
参考资料：
https://yjhjstz.gitbooks.io/deep-into-node/chapter8/chapter8-1.html
http://www.runoob.com/nodejs/nodejs-stream.html
https://nodejs.org/api/stream.html
```
