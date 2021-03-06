相关源码见：[koa-demo](https://github.com/hanekaoru/koa-demo)

----

## koa 

```koa``` 是 ```express``` 的下一代基于 ```Node.js``` 的 ```Web``` 框架

#### Express

```Express``` 是第一代最流行的 ```Web``` 框架，它对 ```Node.js``` 进行了封装：

```js
var express = require("express");

var app = express();

app.get("/", function (req, res) {
    res.send("hello world");
})

app.listen(3000, function () {
    console.log("Example app listening on port 3000");
})
```

虽然 ```Express``` 的 ```API``` 很简单，但是它是基于 ```ES5``` 的语法，要实现异步代码，只有使用 **回调**，如果异步嵌套层次过多：

```js
app.get("/test", function (req, res) {
    fs.readFile("/file1", function (err, data) {
        if (err) {
            res.status(500).send("read file1 error");
        }
        fs.readFile("/file2", function (err, data) {
            if (err) {
                res.status(500).send("read file2 error");
            }
            res.type("text/plain");
            res.send(data);
        });
    });
});
```

虽然可以使用例如 ```async``` 这样的库来组织代码，但是利用回调来处理异步还是太麻烦


## koa 1.0

和 ```Express``` 相比，```koa 1.0``` 使用 ```generator``` 来实现异步：

```js
var koa = require("koa");
var app = koa();

app.use("./test", function * () {
    
    yield doReadFile1();

    var data = yield doReadFile2();

    this.body = data;
    
})

app.listen(3000)
```

利用 ```generator``` 实现异步是简单了不少，但是 ```generator``` 的本意并不是异步（```promise``` 才是）

```ES7```（草案，未来标准） 中引入了新的关键字 ```async``` 和 ```await```，可以轻松把 ```function``` 变为异步模式：

```js
async function () {
    var data = await fs.read("./test")
}
```


## koa 2

和 ```koa 1``` 相比，```koa 2``` 完全使用 ```Promise``` 并配合 ```async``` 来实现异步

```js
app.use(async (ctx, next) => {

    await next();

    var data = await doReadFile();

    ctx.response.type = "text/plain";

    ctx.response.body = data;
    
});
```

#### 创建 koa2 工程

```js
// 导入 koa 和 koa 1.x 不同，在 koa2 中，我们导入的是一个 class，因此用大写的 Koa 来表示
const Koa = require("koa");

// 创建一个 Koa 对象表示 web app 本身
const app = new Koa();

// 对于任何请求，app 将调用该异步函数处理请求
app.use(async (ctx, next) => {

    await next();

    // 设置 response 的 Content-Type
    ctx.response.type = "text/html";

    // 设置 response 的内容
    ctx.response.body = "<h1>hello world</h1>"

})

// 箭头 3000 端口
app.listen(3000);

console.log("app started at port 3000")
```

* 关于 ```ctx``` 参数，是由 ```koa``` 传入的封装了 ```request``` 和 ```response``` 的变量，我们可以通过它们来访问 ```request``` 和 ```response```

* 关于 ```next``` 参数，是 ```koa``` 传入的将要处理的下一个异步函数

* ```node``` 版本过低的话，会执行报错

上面的异步函数中，我们首先调用 ```await next()``` 来处理下一个异步函数，然后，设置 ```response``` 的 ```Content-Type``` 内容

由 ```async``` 标记的函数称为 异步函数，在异步函数中，可以用 ```await``` 调用另一个 异步函数（这两个关键字将在 ```ES7``` 中引入）



#### koa 中间件（middleware）

在来看看 ```koa``` 执行的核心逻辑代码：

```js
app.use(async (ctx, next) => {

    await next();

    ctx.response.type = "text/html";

    ctx.response.body = "<h1></h1>"
    
})
```

每收到一个 ```http``` 请求，```koa``` 就会调用通过 ```app.use()``` 注册的 ```async``` 函数，并传入 ```ctx``` 和 ```next``` 参数，我们可以对 ```ctx``` 操作，并设置返回内容



#### 为什么需要调用 await next()

原因在于 ```koa``` 把很多 ```async``` 函数组成一个处理链，每个 ```async``` 函数都可以做一些自己的事情，然后使用 ```await next()``` 来调用下一个 ```async``` 函数

例如，可以使用下面 3 个 ```middleware``` 组成处理链，依次打印日志，记录处理事件，输出 ```html```：

```js
app.use(async (ctx, next) => {
    console.log(`${ctx.request.method} ${ctx.request.url}`);
    await next();
})

app.use(async (ctx, next) => {
    const start = new Date().getTime();
    await next();
    const ms = new Date().getTime() - start;
    console.log(`Time: ${ms}ms`)
})

app.use(async (ctx, next) => {
    await next();
    ctx.response.type = "text/html"
    ctx.response.body = "<h1></h1>"
})
```

```middleware``` 的顺序很重要，也就是调用 ```app.use()``` 的顺序决定了 ```middleware``` 的顺序

需要注意的是：如果其中一个 ```middleware``` 没有调用 ```await next()```，那么后续的 ```middleware``` 将不会在继续往下执行了

例如一个检测用户权限的 ```middleware```：

```js
// 决定是否继续处理请求，还是直接返回 403 错误
app.use(async (ctx, next) => {
    if (await checkUserPermission(ctx)) {
        await next();
    } else {
        ctx.response.status = 403;
    }
})
```

```ctx``` 对象还有一些简写的方法：

* ```ctx.url``` 相当于 ```ctx.reaponse.url```

* ```ctx.type``` 相当于 ```ctx.response.type```


## 处理 URL

正常情况下，我们应该对不同的 ```URL``` 调用不同的处理函数，这样才能返回正确的结果：

```js
app.use(async (ctx, next) => {
    if (ctx.request.path == "/") {
        ctx.response.body = "index.page";
    } else {
        await next();
    }
})

app.use(async (ctx, next) => {
    if (ctx.request.path == "/test") {
        ctx.response.body = "test.page"
    } else {
        await next();
    }
})

app.use(async (ctx, next) => {
    if (ctx.request.path ==  "error") {
        ctx.response.body = "error.page"
    } else {
        await next();
    }
})
```

这样写是可以运行的，但是好像有点蠢


#### koa-router

为了处理 ```URL```，我们引入 ```koa-router``` 这个 ```middleware```，它负责处理 ```URL``` 映射：

```js
const Koa = require("koa");

// 需要注意的是，koa-router 返回的是函数（直接拿到调用后的结果）
// 相当于 
// const fn_router = require("koa-router");
// const touter = fn_router();
const router = require("koa-router")();

const app = new Koa();

// log request URL
app.use(async (ctx, next) => {
    console.log(`process ${ctx.request.method} ${ctx.request.url}...`);
    await next();
})

// add url-router
router.get("/hello/:name", async (ctx, next) => {
    var name = ctx.params.name;
    ctx.response.body = `<h1>hello ${name}</h1>`
    await next();
})

router.get("/", async (ctx, next) => {
    ctx.response.body = "<h1>Index</h1>"
})

// add router middleware
app.use(router.routes());

app.listen(3000);

console.log("app start at port 3000")
```


## 处理 POST 请求

使用 ```router.get("/path", async fn)``` 处理的是 ```get``` 请求，如果要处理 ```POST``` 请求，可以使用 ```router.post("/path" async fn)```

使用 ```POST``` 请求处理 ```URL``` 的时候，我们会遇到一个问题，```POST``` 请求通常会发送一个表单，或者 ```JSON```，它作为 ```request``` 的 ```body``` 发送，但无论是 ```Node.js``` 提供的原始 ```request``` 对象，还是 ```koa``` 提供的 ```request（ctx）```对象，都**不提供**解析 ```request``` 的 ```body``` 的功能

所以我们需要引入另外一个中间件 ```koa-bodyparser``` 来解析原始的 ```request``` 请求，然后把解析后的参数绑定到 ```ctx.request.body``` 中

```js
// 然后在 app.js 中引入 koa-bodyparser
const bodyParser = require("koa-bodyparser");

// 然后在合适的位置调用
app.use(bodyParser());
```

由于 ```middleware``` 的顺序很重要，这个 ```koa-bodyparser``` 必须在 ```router``` 之前被注册到 ```app``` 对象上面

现在我们就可以处理 ```post``` 请求了：

```js
// 相关依赖
const Koa = require("koa");
const router = require("koa-router")();
const bodyParser = require("koa-bodyparser");

const app = new Koa();

// 必须在 router 之前被注册到 app 对象上
app.use(bodyParser());

// 生成表单，提交到 /signin 下
router.get("/", async (ctx, next) => {
    ctx.response.body = `<h1>Index</h1>
        <form action="/signin" method="post">
            <p>Name: <input name="name" value="koa"></p>
            <p>Password: <input name="password" type="password"></p>
            <p><input type="submit" value="Submit"></p>
        </form>`;
})

// 处理表单请求
router.post("/signin", async (ctx, next) => {

    // 如果该字段不存在，默认值设置为 ""
    var name = ctx.request.body.name || "",
        password = ctx.request.body.password || "";
    console.log(`signin with name: ${name}, password: ${password}`);

    if (name == "koa" && password == "123") {
        ctx.response.body = `<h1>hello, ${name}</h1>`
    } else {
        ctx.response.body = `try again`
    }
    
})


app.use(router.routes());

app.listen(3000);

console.log("app start at port 3000")
```

类似的 ```put```，```delete```，```head``` 请求也可以由 ```router``` 来处理


#### 重构

现在，我们可以处理不同的 ```URL``` 了，但是所有的逻辑都堆在 ```app.js``` 中，随着 ```URL``` 越来越多，```app.js``` 就会越来越长

我们可以让 ```URL``` 处理函数集中到某个 ```js``` 文件，然后让 ```app.js``` 自动导入所有处理 ```URL``` 的函数，这样，代码一分离，逻辑就显得清楚了：

```js
// index.js 
var fn_index = async (ctx, next) => {
    ctx.response.body = `<h1>Index</h1>
        <form action="/signin" method="post">
            <p>Name: <input name="name" value="koa"></p>
            <p>Password: <input name="password" type="password"></p>
            <p><input type="submit" value="Submit"></p>
        </form>`;
}

var fn_signin = async (ctx, next) => {
    var name = ctx.request.body.name || "",
        password = ctx.request.body.password || "";
    console.log(`signin with name: ${name}, password: ${password}`);

    if (name == "koa" && password == "123") {
        ctx.response.body = `<h1>hello, ${name}</h1>`
    } else {
        ctx.response.body = `try again`
    }
}

// 通过 module.exports 把两个 url 处理函数暴露出去
module.exports = {
    "GET /": fn_index,
    "POST /signin": fn_signin 
}
```



```js
// hello.js
var fn_hello = async (ctx, next) => {
    var name = ctx.params.name;
    ctx.response.body = `<h1>hello, ${name}</h1>`
}

module.exports = {
    "GET /hello/:name": fn_hello
}
```



```js
// app.js
// 修改 app.js 让它自动扫描 controllers 目录，找到所有 js 文件导入，然后注册每个 url

// 先导入 fs 模块，然后使用 readdirSync 列出文件
var fs = require("fs");

// 这里可以使用 async，因为启动的时候只运行一次，不存在性能问题
var files = fs.readdirSync(__dirname + "/controllers");

// 过滤 .js 文件
var js_file = files.filter( (f) => {
    return f.endsWith(".js");
})

// 处理每个 js 文件
for (var f of js_file) {

    console.log(`process controllers: ${f} ...`)

    // 导入 js 文件
    let mapping = require(__dirname + "/controllers/" + f);

    for (var url in mapping) {

        if (url.startsWith("GEt ")) {

            // 如果 url 类似 "GET xxx"
            var path = url.substring(4);
            router.get(path, mapping[url]);
            console.log(`register URL mapping: GET ${path}`);

        } else if (url.startsWith("POST ")) {

            // 如果 URL 类似 "POST xxx"
            var path = url.substring(5);
            router.post(path, mapping[url]);
            console.log(`register URL mapping: POST ${path}`);

        } else {

            // 无效的 url
            console.log(`invalid URL: ${url}`);
            
        }
    }
}
```


#### Controller Middleware

现在我们来重构和整合上面的代码，把扫描 ```controllers``` 目录和创建 ```router``` 的代码从 ```app.js``` 中提取出来，作为一个简单的 ```middleware``` 来使用，命名为 ```controller.js```

这样一来，我们的 ```app.js``` 中的代码已经十分精简了，所有处理 ```URL``` 的函数按功能组存放在 ```controllers``` 目录

```js
// app.js
const Koa = require("koa");
const bodyParser = require("koa-bodyparser");
const controller = require("./controller");
const app = new Koa();

app.use(async (ctx, next) => {
    console.log(`Process ${ctx.request.method} ${ctx.request.url}...`);
    await next();
});

app.use(bodyParser());

app.use(controller());

app.listen(3000);
console.log("app started at port 3000...");
```

```js
// controller.js
const fs = require("fs");

function addMapping (router, mapping) {
    for (var url in mapping) {
        if (url.startsWith("GET ")) {
            var path = url.substring(4);
            router.get(path, mapping[url]);
            console.log(`register URL mapping: GET ${path}`);
        } else if (url.startsWith("POST ")) {
            var path = url.substring(5);
            router.post(path, mapping[url]);
            console.log(`register URL mapping: POST ${path}`);
        } else {
            console.log(`invalid URL: ${url}`);
        }
    }
}

function addControllers (router, dir) {
    fs.readdirSync(__dirname + "/" + dir).filter((f) => {
        return f.endsWith(".js")
    }).forEach((f) => {
        console.log(`process controlles: ${f}...`)
        let mapping = require(__dirname + "/" + dir + "/" + f)
        addMapping(router, mapping)
    })
}

module.exports = function (dir) {
    let controllers_dir = dir || "controllers",
        router = require("koa-router")();
    addControllers(router, controllers_dir);
    return router.routes();
}
```

```js
// controller / controller.js
var fn_index = async (ctx, next) => {
    ctx.response.body = `<h1>Index</h1>
        <form action="/signin" method="post">
            <p>Name: <input name="name" value="koa"></p>
            <p>Password: <input name="password" type="password"></p>
            <p><input type="submit" value="Submit"></p>
        </form>`;
};

var fn_signin = async (ctx, next) => {
    var
        name = ctx.request.body.name || "",
        password = ctx.request.body.password || "";
    console.log(`signin with name: ${name}, password: ${password}`);
    if (name === "koa" && password === "12345") {
        ctx.response.body = `<h1>Welcome, ${name}!</h1>`;
    } else {
        ctx.response.body = `<h1>Login failed!</h1>
        <p><a href="/">Try again</a></p>`;
    }
};

module.exports = {
    "GET /": fn_index,
    "POST /signin": fn_signin
};
```

```js
// controller / hello.js
var fn_hello = async (ctx, next) => {
    var name = ctx.params.name;
    ctx.response.body = `<h1>hello, ${name}</h1>`
}

module.exports = {
    "GET /hello/:name": fn_hello
}
```

详细的源码可以见：[koa-post](https://github.com/hanekaoru/koa-demo/tree/master/koa-post-reconsitution)

#### 关于 controller.js 中最后返回 router.routes()

之前我们相关 ```url``` 处理都放在 ```app.js``` 里面，使用的时候是：

```js
router.get(...)
router.post(...)

app.use(router.routes());
```

而在重构之后，我们在 ```app.js``` 中使用的话就是：

```js
app.use(controller());
```

相关的 ```router.get()``` 和 ```router.post()``` 已经在 ```controller.js``` 里面处理完成了

然后 ```contorller()``` 将之后的 ```router.routes()``` 返回给 ```app.js```





## Nunjucks

本质上，我们只需要构造这样一个函数：

```js
function render (view, model) {
    // ...
}
```

```view``` 是模版的名称（视图），```model``` 就是数据，在 ```js``` 中，它就是一个简单的 ```Object```，```render``` 函数返回一个字符串，就是模版的输出

```js
const nunjucks = require("nunjucks");

function createEnv (path, opts) {

    var autoescape = opts.autoescape && true,
        noCache = opts.noCache || false,
        watch = opts.watch || false,
        throwOnUndefined = opts.throwOnUndefined,

        env = new nunjucks.Environment(
            new nunjucks.FileSystemLoader("views", {
                noCache: noCache,
                watch: watch,
            }), {
                autoescape: autoescape,
                throwOnUndefined: throwOnUndefined
            });

    if (opts.filters) {
        for (var f in opts.filters) {
            env.addFilter(f, opts,filters[f]);
        }
    } 
    
    return env;
}

var env = createEnv("views", {
    watch: true,
    filters: {
        hex: function (n) {
            return "0x" + n.toString(16);
        }
    }
})
```

变量 ```env``` 就表示 ```Nunjucks``` 模版引擎对象，它有一个 ```render(view, model)``` 方法，正好传入 ```view``` 和 ```model``` 两个参数，并返回字符串

我们使用 ```autoescape = opts.autoescape && true``` 这样的代码给每个参数加上默认值，最后使用 ```new nunjucks.FileSystemLoader("views")``` 创建一个文件系统加载器，从 ```views``` 目录读取模版

比如：

```js
<h1>hello, {{ name }}</h1>
```

我们可以来渲染它：

```js
var s = env.render("hello.html", { name: "小明" });

console.log(s);
```

源码见：[koa-nunjucks](https://github.com/hanekaoru/koa-demo/tree/master/koa-nunjucks)





## MVC 

当用户通过浏览器请求一个 ```url``` 的时候，```koa``` 将调用某个异步函数处理该 ```url```，在这个异步函数内部，我们用一行代码：

```js
ctx.render("index.html", { name: "zhangsan" })
```

基本流程如下：

* 浏览器请求（GET /zhangsan） name = "zhangsan"

* app.js（接收处理请求）

```js
// GET /:name
async (ctx, next) => {
    ctx.render("index.html", {
        name: ctx.request.params.name
    });
}
```

* 模版引擎替换变量（{{ name }} ==> "zhangsan"）

```html
<html>
<body>
    <div>hello, {{ name }}</div>
</body>
</html>
```

* 输出

```html
<html>
<body>
    <div>hello, zhangsan</div>
</body>
</html>
```

这就是传说中的 MVC：Model - View - Controller（模型 - 视图 - 控制器）

* Controller 负责业务逻辑，比如检查用户名是否存在，取出用户信息等

* View 包含变量的 ```{{ name }}``` 模版就是视图，负责显示逻辑（最终输出就是 ```html```）

* Model ```model``` 是用来传递给 ```view``` 的，这样在 ```view``` 替换变量的时候，就可以从 ```model``` 中提取相应的数据

详细源码见 [use-koa](https://github.com/hanekaoru/koa-login)

