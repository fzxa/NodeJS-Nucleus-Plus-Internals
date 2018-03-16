### Node Stream内部原理

Stream在平时业务开发时很少用到， 单很多模块都是机遇stream实现的，引用官方文档的解释：

流（stream）在 Node.js 中是处理流数据的抽象接口（abstract interface）。

stream 模块提供了基础的 API 。使用这些 API 可以很容易地来构建实现流接口的对象。

流可以是可读的、可写的，或是可读写的。所有的流都是 EventEmitter 的实例。
