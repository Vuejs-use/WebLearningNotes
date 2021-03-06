使用 webpack 来构建 vue 项目，从头开始搭建，了解一下 webpack 在 vue 项目中的使用

注：webpack 和 vue 的版本均在 2.0 以上

源码见：[webpack-vue](https://github.com/hanekaoru/WebLearningNotes/tree/master/vue/webpack-vue)

## 创建目录

大致目录结构如下：

```js
├── index.html           根本件，最后用于展示 App.vue 组件
├── main.js              入口文件
├── App.vue              vue 文件
├── pacjage.json         工程配置文件（依赖，配置）
├── webpack.config.js    webpack 配置文件  
```

首先新建一个 html 文件，添加一个 id 为 box 的 div

```js
<div id="box"></div>
```

然后在 main.js 文件中添加一点内容，用于最后的渲染

```js
import Vue from "vue"
import App from "./App.vue"

new Vue({
    el: "#box",
    render: h => h(App)
})
```

关于 h => h(App) 见最下方

接下来来安装 webpack 和 webpack-dev-server，再配置一下 webpack，先简单的指定一下入口和出口文件

```js
// webpack.config.js
module.exports = {
	// 入口文件
	entry: "./main.js",

	// 出口文件
	output: {
		filename: "bundle.js",
		// 当前路径
		path: __dirname
	}
};
```

然后就可以在我们的 index.html 中引入出口文件中指定的 js（目录中是不存在这个文件的，webpack 会在执行的时候帮助我们编译出这个文件）

```js
// index.html
<script src="./bundle.js"></script>
```

关于出口文件，如果想指定输出的目录，可以按照下面这样来设置：

```js
output: {

    // 如果目录，如果不存在则会去新建
    path: path.resolve(__dirname, "./src"),

    // 设置静态目录（可以从这个目录下直接进行读取）
    publicPath: "/src/",

    // 文件名称
    filename: "bundle.js"

},
```


## 配置 vue-loader

如果需要解析像 ```App.vue``` 这样的文件，使其变成正常可以访问的代码，我们需要引入 vue-loader 来处理

这个时候我们就可以在 webpack 中配置我们的相关 loader 了（这也是 webpack 最核心的东西），在 webpack.config.js 中添加：

```js
module: {
    // 所有的 loader 都配置在这里
    rules: [
        // 下面这句的意思是 使用 vue-loader 来解析处理以 .vue 结尾的文件
        // 如果多个 loader 的话，中间可以使用 ! 来连接，比如 style!css
        {
            test: /\.vue$/,
            loader: 'vue-loader'
        }
    ]
}
```


配置完 webpack 以后就可以来安装 vue-loader 以及其相关的一些组件了

单单使用 vue-loader 编译写入 .vue 文件是不够的，因为在 .vue 文件当中主要分为三大块，如下所示：

```js
<template>

</template>

<script>

</script>

<style>

</style>
```

每一部分都需要相对应的 loader 来进行处理，这时就需要同时安装以下 loader

```js
// 处理模版
vue-template-compiler  ==> 编译 vue 的 template 部分，（之前使用的是 vue-html-loader） 


// 处理 css，因为 css 分为 .css 文件和行内样式，需要同时加载
css-loader ==> 编译写入 css

vue-style-loader ==> 编译 vue 的样式部分


// 处理 js，关于 babel 部分见下方
vue-hot-reload-api ===> 使 webpack 对 vue 实现热替换
```

可以使用以下命令来进行安装

```js
npm install vue-template-compiler css-loader vue-style-loader vue-hot-reload-api --save-dev
```


## 配置 babel

众所周知，现在浏览器对 ES6 语法的支持还不是很完善，这个时候也就需要使用 babel 来解析 ES6 语法，先来配置 webpack 

```js
module: {
    rules: [
        // 规则和上面类似，需要注意的就是多了一个 exclude
        // 意思是 匹配除了 /node_modules/ 和 bower_components 之外的所有 .js 文件
        // 关于 options 选项，如果在根目录下的 .babelrc 配置了，这里就不需要了，两者是一样的
        // 关于 .babelrc 见最后
        {
            test: /\.js$/,
            exclude: /(node_modules|bower_components)/,
            use: {
                loader: 'babel-loader',
                options: {
                    // 先配置一个前置
                    presets: ["es2015"],
                    // 还需要一个插件来使每次改变的时候，实时编译
                    plugins: ["transform-runtime"]
                }
            }
        }

    ]
}
```

babel 需要的相关组件如下：

```js
babel-loader  ==> 主程序

babel-core  ==> 编译核心（语法层面）

babel-plugin-transform-runtime  ==> 实时编译插件

babel-perset-es2015  ==> es6 语法

babel-runtime  ==> 上面几个程序需要这个组件来运行（babel 执行环境）
```

最后来汇总一下所有涉及到的依赖：

```js
vue-loader  ==> 编译写入 .vue 文件

// 处理 .vue 中的模版
vue-template-compiler  ==> 编译 vue 的 template 部分，（之前使用的是 vue-html-loader） 


// 处理 css，因为 css 分为 .css 文件和行内样式，需要同时加载
css-loader ==> 编译写入 css

vue-style-loader ==> 编译 vue 的样式部分


// 处理 js
vue-hot-reload-api ===> 使 webpack 对 vue 实现热替换

// 使用 babel 来处理 es6 语法
babel-loader  ==> 主程序

babel-core  ==> 编译核心（语法层面）

babel-plugin-transform-runtime  ==> 实时编译插件

babel-perset-es2015  ==> es6 语法

babel-runtime  ==> 上面几个 babel 程序需要这个组件来运行（babel 执行环境）


// 处理文件和图片
file-loader  ==> 用于打包文件和图片，默认情况下生成文件的文件名是文件名与 md5 哈希值的组合

// webpack
webpack  ==> webpack 主程序

webpack-dev-server  ==> webpack 执行环境


// vue
vue  ==> 这个就不用多说了


// 可选
url-loader ==> 实际上是对 file-loader 的封装（见最后）

webpack-merge  ==> 开发环境和生产环节的 webpaak 配置文件的配置合并
```



## 启动

安装和配置完成以后，这个时候就可以来启动程序了

但是如果想在命令行直接使用类似 ```npm run dev``` 命令的话，就需要配置一下 package.json 文件

在 scripts 标签下设置：

```js
// 设置 scripts 即可
// --inline 自动刷新
// --hot 热载
// --port 设定端口（在端口被占用的情况下可以利用这个参数来指定，默认的为 8080）
"scripts": {
    "dev": "webpack-dev-server --inline --hot"
}
```

然后就可以使用 ```npm run dev``` 命令来启动服务了，如果在控制台中看到：

```js
Project is running at http://localhost:8080/
```

这个时候就表明启动成功了，可以访问 http://localhost:8080/ 来查看效果了 

如果配置全部正确的话，就可以在浏览器中看到对应的内容了，修改内容直接保存，浏览器也会自动跟着刷新

## 附录：关于 h => h(App)

在 vuejs 中，可以把 h 函数仅是作为 createElement 函数的缩写，可以将上述代码理解为

```js
render: function (createElement){
    return createElement(app);
}
```

render 方法：

```js
render: function (createElement) {

    return createElement(

        // tag name 标签名称
        "h" + this.level,

        // 子组件中的阵列
        this.$slots.default
    )

}
```

也是 2.0 新增的函数，可以直接给绑定节点渲染一个 vue 组件，如果在 1.x 的版本下，就应该使用

```js
new Vue({
    el: "#app",
    components: { App }
});

// html
<div id="app">
    <app></app>
</div>
```




## 附录：关于 .babelrc

和 .bowerrc 是一样的原理，都是用来配置文件，只需要在根目录新建一个名为 .bowerrc 文件，然后添加：

```js
{
  "presets": ["es2015", "stage-0"],
  "plugins": ["transform-runtime"]
}
```

就可以达到和上面一样的效果


## 附录：关于 url-loader

url-loader 是对 file-loader 的上层封装，比如可以在 webpack 中对图片的加载器进行配置

```js
{
    test: /\.(png|jpg)$/,
    loader: "url-loader?limit=8192"
}
```

这样一来，在小于 8K 的图片将直接以 base64 的形式内联在代码中，可以减少一次 http 请求

所以在处理一般小图片的时候就可以使用 url-loader 来转为 base64，其他情况照常使用 file-loader

