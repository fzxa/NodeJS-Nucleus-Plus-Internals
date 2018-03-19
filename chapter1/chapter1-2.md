### Node Stream模块分析

Stream在平时业务开发时很少用到， 单很多模块都是机遇stream实现的，引用官方文档的解释：

流（stream）在 Node.js 中是处理流数据的抽象接口（abstract interface）。

stream 模块提供了基础的 API 。使用这些 API 可以很容易地来构建实现流接口的对象。

流可以是可读的、可写的，或是可读写的。所有的流都是 EventEmitter 的实例。


#### 为什么应该使用流

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

这样减少了很大的服务器压力

```js
var http = require('http');
var fs = require('fs');

var server = http.createServer(function (req, res) {
    let stream = fs.createReadStream(__dirname + '/data.txt');//创造可读流
    stream.pipe(res);//将可读流写入response
});
server.listen(8000);
```
