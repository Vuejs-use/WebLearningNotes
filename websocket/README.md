源码见：[WebSocket-demos](https://github.com/hanekaoru/WebSocket-demos)

----

## WebSocket

WebSocket 是 HTML5 新增的协议，它的目的是在浏览器和服务器之间建立一个不受限的双向通信的通道，比如说服务器可以在任意时刻发送消息给浏览器

Websocket 协议本质上是一个基于 TCP 的协议，它由通信协议和编程 API 组成，WebSocket 能够在浏览器和服务器之间建立双向连接，以基于事件的方式，赋予浏览器事实通信的能力，既然是双向通信，就意味着服务器端和客户端可以同时发送并响应请求，而不再像 HTTP 的请求和响应

为了建立一个 WebSocket 连接，客户端浏览器首先要向服务器发起一个 HTTP 请求，这个请求和通常的 HTTP 请求不同，包含了一些附加头信息，其中附加头信息 "Upgrade: WebSocket" 表明这是一个申请协议升级的 HTTP 请求

服务器端解析这些附加的头信息然后产生应答信息返回给客户端，客户端和服务器端的 WebSocket 连接就建立起来了，双方就可以通过这个连接通道自由的传递信息，并且这个连接会持续存在直到客户端或者服务器端的某一方主动的关闭连接

## WebSocket API

一个简单的实例

大致流程为：打开一个连接，为连接创建事件监听器，断开连接，消息时间，发送消息返回到服务器，关闭连接

```js
// 创建一个 Socket 实例
var socket = new WebSocket("ws://localhost:8080");
 
// 打开 Socket
socket.onopen = function (event) {
 
    // 发送一个初始化消息
    socket.send("socket init")
 
    // 监听消息
    socket.onmessage = function (event) {
        console.log("Message listener")
    }
 
    // 监听 socket 的关闭
    socket.onclose = function (event) {
        console.log("closed")
    }
 
    // 关闭
    socket.close()
}

```

* ws 表示 WebSocket 协议，参数为 url（以 ws 开头）

* onopen，onclose，onmessage 方法把事件连接到 Socket 实例上

* onmessage 事件提供了一个 data 属性，它可以包含消息的 body 部分（消息的 body 部分必须为一个字符串，可以进行 序列化/反序列化，以便传递更多的数据）



## WebSocket 与 Socket.IO

Socket.IO 是 node.js 的一个模块，它是通过 WebSocket 进行通信的一种简单方式，Socket.IO 使用检测功能来判断是否建立 WebSocket 连接，或者是 AJAX long-polling 连接，或 Flash 等。可快速创建实时的应用程序

一个简单的示例：

```js
// 客户端 index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Socket.io</title>
</head>
<body>
    <h1>I Am Socket.IO</h1>
    <script src="../socket.io.js"></script>
    <script>
        var socket=io.connect("ws://localhost:8000");
    </script>
</body>
</html>
// 服务端 index.js
var http = require("http");
var fs = require("fs");
 
var sever = http.createServer(function (req, res) {
    fs.readFile("./index.html", function (err, data) {
        res.writeHead(200, { "Content-Type": "text/html" });
        res.end(data, "utf-8");
    });
}).listen(3000);
 
// 为了在服务器上加入 Socket.io 的功能，必须将 Socket.IO 库包括进来，而后附加到服务器上
var io = require("socket.io").listen(sever);
 
// 在启动了服务器的 Socket.io 之后，用于初始化
io.socket.on("connection", function (socket) {
 
    console.log("user conneted");
 
    socket.on("disconnect", function () {
        console.log("user disconnet");
    });
 
});
```

向服务器发送数据到客户端

```js
io.sockets.on("connection", function (socket) {
    // 向客户端发送消息
    socket.emit("message", { text:"you have connected" });
});
```

只要客户端连接，它就将数据发送给每一个新的客户端，而如果想给当前所有的客户端都发送消息，则需要发送广播消息

```js
io.sockets.on("connection", function (socket) {
 
    // 单个客户端发送消息
    socket.emit("message", { text: "A new user has connected" });
 
    // 广播消息给客户端
    socket.broadcast.emit("massage", { text: "A new user has connected" });
});
```

接下来需要做的就是客户端先连接 Socket.io 服务器，然后侦听在 "message" 事件上接收的数据，然后做出响应

```js
var socket = io.connect("ws://localhost:8080");
socket.on("message", function (data) {
    console.log(`${data.text}`);
});
```