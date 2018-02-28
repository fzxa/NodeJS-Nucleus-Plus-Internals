### NodeJS源码解析 - HTTP Server

上一篇文章提到了hello world中的server ,本篇文章会对server的运行机制进行研究
以下是启动一个server最简单的代码例子：

```
http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World\n');
}).listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

#### Server

