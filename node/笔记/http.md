#### request 对象

* httpVersion  使用的 http 协议的版本

* headers  请求头信息中的数据

* url  请求的地址

* method  请求方式

#### response 对象

* write(chunk, [encoding])  发送一个数据块到响应正文中

* end([chunk], [encoding])  当所有的正文和头信息发送完毕后调用该方法告诉服务器数据已经全部发送完毕了，这个方法在每次完成信息发送以后必须调用，并且是在最后调用（否则浏览器会一直处于等待状态）

* statusCode  用来设置返回的状态码

* setHeader(name, value)  设置返回头信息

* writeHead(statusCode, [reasonPhrase], [headers])  写入头信息，这个方法只能在当前请求中使用一次，并且必须在 response.end() 方法之前调用

#### url 模块

我们可以使用 node 提供的核心模块 url 来处理用户的请求路径

```js
var urlStr = url.parse("http://www.baidu.com/a/index.html?b=2#p=1");
console.log(urlStr);

// -------------------------------

server.on("request", function (req, res) {

    // req.url 即为 访问路径
    // ? 后面的部分为一个 query string

    var urlStr = url.parse(req.url);

    swich (urlStr.pathname) {
        case "/":
            // 首页
            res.write(200, {"Content-Type", "text/html;charset=utf-8"})
            res.end("<h1>首页</h1>")
            break;
        
        case "/user":
            // 用户页
            res.write(200, {"Content-Type", "text/html;charset=utf-8"})
            res.end("<h1>用户页</h1>")
            break;

        default: 
            // 其他情况
            res.write(404, {"Content-Type", "text/html;charset=utf-8"})
            res.end("<h1>没找到</h1>")
            break;
    }
})
```

#### 使用 fs 模块实现表现分离

大致目录结构如下：

```js
├── app.js
└── static
    ├── index.html
    ├── user.html
    └── err.html
```

```js
// ...

var htmlDir = __dirname + "/static/";

server.on("request", function (req, res) {

    var urlStr = url.parse(req.url);

    swich (urlStr.pathname) {
        case "/":
            // 首页
            sendData(htmlDir + "index.html", req, res)
            break;
        
        case "/user":
            // 用户页
            sendData(htmlDir + "user.html", req, res)
            break;

        default: 
            // 其他情况
            sendData(htmlDir + "err.html", req, res)
            break;
    }
})

function sendData (file, req, res) {
    fs.readFile(file, function (err, data) {
        if (err) {
            res.write(404, {"Content-Type", "text/html;charset=utf-8"})
            res.end("<h1>没找到</h1>")
        } else {
            res.write(200, {"Content-Type", "text/html;charset=utf-8"})
            res.end(data)
        }
    })
}

// ...
```

----

----

同样的，也可以利用上面的方式来处理 get 和 post 请求

这其中会用到 queryString.parse() 方法

```js
queryString.parse(str, [sep], [eq], [options])
```

用来将一个 query string 反序列化为一个对象，可以选择是否覆盖默认的分隔符（&）和分配符（=）

```js
username=123&password=123  ===>  { username: 123, password: 123 }
```

get 请求的处理方法与之前的类似

```js
// ...

swich (urlStr.pathname) {
    case "/login":
        // 登录
        sendData(htmlDir + "index.html", req, res)
        break;
    
    // ...
}

// ...
```

post 请求需要特别说明一下

post 发送的数据会被写入缓冲区中，需要通过 request 的 data 事件和 end 事件来进行数据的处理

```js
// ...

swich (urlStr.pathname) {
    case "/login":
        if (req.method.toUpperCase() == "POST") {
            
            var str = "";
            
            req.on("data", function (chunk) {
                str += chunk;
            })

            req.on("end", function () {
                console.log(queryString(str))
            })
        }
    
    // ...
}

// ...
```