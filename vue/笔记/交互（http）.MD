## 交互（ajax）

如果 vue 想做交互，需要引入另外一个库 vue-resouce

#### get

获取一个普通文本数据

```js
new Vue({
    el: "#app",
    data: {},
    methods: {
        get: function () {
            // 返回的是一个 promise 对象
            this.$http.get("1.txt").then(function(res) {
                console.log(res.data)
            })
        }
    }
})

// ==========================================================

// html
<div id="app">
    <input type="button" value="按钮" @click="get()">
</div>
```

使用 get 向服务器发送请求

同上面类似，data 是作为第二个参数传入

```js
// ...

methods: {
    get: function () {
        this.$http.get("url?page=", {
            params: {
                page: 10
            }
        }).then(function(res) {
            console.log(res.data)
        })
    }
}

// ...
```

使用 post 向服务器发送数据

也类似，不同的是需要添加第三个参数（option）来做标识，才能把 post 数据提交出去

```js
methods: {
    get: function () {
        this.$http.post("url", {
            a: 123,
            b: 456
        }, {
            emulateJSON: true
        }).then(function(res) {
            console.log(res.data)
        })
    }
}
```


使用 jsonp 跨域请求数据

有几点需要注意，如果接口默认的回调函数名称为 callback，如下所示，直接使用即可

```js
methods: {
    get: function () {
        this.$http.jsonp("www.123.com/search=123&callback=abc", {
            search: abc
        }).then(function(res) {
            console.log(res.data)
        })
    }
}
```

但是如果回调函数名称不是callback，那就需要指定第三个参数，用来指定 callback 的名称（vue 默认的回调函数名称为 callback）


```js
methods: {
    get: function () {
        this.$http.jsonp("www.123.com/search=123&cb=abc", {
            search: abc
        }, {
            jsonp: "cb"
        }).then(function(res) {
            console.log(res.data)
        })
    }
}
```