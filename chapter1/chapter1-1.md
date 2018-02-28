### NodeJS源码解析 - HTTP Server模块

上一篇文章提到了hello world中的http server 

http是nodejs中重要的模块之一，有必要了解它的运行原理

```
var http = require('http');
http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World\n');
}).listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

#### Server
1. 打开node-v8.9.3/lib/http.js 

首先引入的是http模块,模块抛出公共方法调用createServer实际上是返回Server实例，

createServer里面的回调函数（参数requestListener）

直接作为了Server的参数requestListener,而这个Server实际上是require('_http_server')
```
'use strict';

const agent = require('_http_agent');
const { ClientRequest } = require('_http_client');
const common = require('_http_common');
const incoming = require('_http_incoming');
const outgoing = require('_http_outgoing');
//引入私有_http_server模块
const server = require('_http_server');

const { Server } = server;
//创建server, 将回调函数作为参数
function createServer(requestListener) {
  return new Server(requestListener);
}

function request(options, cb) {
  return new ClientRequest(options, cb);
}

function get(options, cb) {
  var req = request(options, cb);
  req.end();
  return req;
}

module.exports = { 
  _connectionListener: server._connectionListener,
  METHODS: common.methods.slice().sort(),
  STATUS_CODES: server.STATUS_CODES,
  Agent: agent.Agent,
  ClientRequest,
  globalAgent: agent.globalAgent,
  IncomingMessage: incoming.IncomingMessage,
  OutgoingMessage: outgoing.OutgoingMessage,
  Server,
  ServerResponse: server.ServerResponse,
  createServer, //暴露createServer方法
  get,
  request
};
```

打开文件node-v8.9.3/lib/_http_server.js 260行

实际上是为这个requestListener函数与'request'事件绑定到了一起
   而'request '是方法parserOnIncoming里面抛出的一个事件
```
function Server(requestListener) {
  if (!(this instanceof Server)) return new Server(requestListener);
  net.Server.call(this, { allowHalfOpen: true }); 

  if (requestListener) {
    this.on('request', requestListener);
  }

  // Similar option to this. Too lazy to write my own docs.
  // http://www.squid-cache.org/Doc/config/half_closed_clients/
  // http://wiki.squid-cache.org/SquidFaq/InnerWorkings#What_is_a_half-closed_filedescriptor.3F
  this.httpAllowHalfOpen = false;

  this.on('connection', connectionListener);

  this.timeout = 2 * 60 * 1000;
  this.keepAliveTimeout = 5000;
  this._pendingResponseData = 0;
  this.maxHeadersCount = null;
}
util.inherits(Server, net.Server);
```
