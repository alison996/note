### TCP传输控制协议

#### 1、OSI模型
由七层组成，从上到下分别是：
	1）应用层（HTTP/SMTP/IMAP等）；
	2）表示层（加密/解密等）；
	3）会话层（通信连接/维持会话）；
	4）传输层（TCP/UDP）；
	5）网络层（IP）；
	6）链路层（网络特有的链路接口）；
	7）物理层（网络物理硬件）；

-----
#### 2、TCP协议
`TCP协议`是`面向连接`的`传输层协议`，其特点是在传输前需要`3次握手`形成会话。
	1）客户端请求连接；
	2）服务端响应；
	3）客户端传输数据；
当会话形成后，服务端与客户端之间才能互相发送数据。在创建会话的过程中，由`服务端`和`客户端`分别提供一个`套接字`，共同形成一个连接。

-----
#### 3、创建TCP服务端
```javascript
// 创建TCP服务端
const net = require('net');

const server = net.createServer(function(socket) {
  // data事件
  socket.on('data', function(data) {
    socket.write(Buffer.from('welcome'));
  });

  // end事件
  socket.on('end', function() {
    console.log(Buffer.from('disconnect'));
  });

  // 向客户端发送数据
  socket.write(Buffer.from('hello'));
});

// 监听端口
server.listen(8124, function() {
  console.log('TCP server is running at port 8124.');
});
```

-----
#### 4、创建TCP客户端
```javascript
// 创建TCP客户端
const net = require('net');

// 连接TCP服务端
const client = net.connect({ port: 8124 }, function() {
  console.log('clent connected');
  client.write(Buffer.from('Granted'));
});

client.on('data', function(data) {
  console.log('data', data.toString());
  // 关闭连接
  client.end();
});

client.on('end', function() {
  console.log('disconnected');
});
```

-----
#### 5、事件
##### 5.1 服务器事件
```javascript
const net = require('net');
// 创建服务端实例
const server = net.createServer();

// 监听connection事件
server.on('connection', function(socket) {
    // ...此处省略
});

// 监听listening事件 在调用server.listen()绑定端口后触发
server.on('listening', function() {
    // ...此处省略
});

// 监听close事件 调用server.close()后，等待所有连接断开后触发。与end有区别
server.on('close', function() {
    // ...此处省略
});

// 监听error事件 服务器发生异常时触发
server.on('error', function() {
    // ...此处省略
});

server.listen(8124, function() {
    console.log('TCP server is running at port 8124.');
});
```
##### 5.2 连接事件
```javascript
// 创建TCP客户端
const net = require('net');

// 连接TCP服务端
const client = net.connect({ port: 8124 }, function() {
  console.log('clent connected');
  client.write(Buffer.from('Granted'));
});

// 监听connect事件 用于客户端，当套接字与服务端连接成功时触发
client.on('connect', function() {
  // ...此处省略
});

// 监听data事件 一端调用write()发送数据时，另一端触发data事件
client.on('data', function(data) {
  // ...此处省略
});

// 监听drain事件 任意端调用write()发送数据时，当前端触发
client.on('drain', function(drain) {
  // ...此处省略  ps：自己写了demo没触发，回头再查查资料
});

// 监听timeout事件 连接被闲置时触发
client.on('timeout', function() {
    // ...此处省略
});

// 监听end事件 任意端发送FIN数据时触发
client.on('end', function() {
  // ...此处省略
});

// 监听close事件 套接字完全关闭时触发
client.on('close', function() {
    // ...此处省略
});

// 监听error事件 发生异常时触发
client.on('error', function() {
    // ...此处省略
});
```

-----
#### 6、Nagle算法
`Node`中默认启用`Nagle算法`，当`缓冲区`的数据达到`一定数量`或者`一定时间`后才将其发出，小的数据包将会被`Nagle算法`合并，导致数据可能被`延迟发送`。调用`socket.setNoDelay(true)`关闭`Nagle算法`。