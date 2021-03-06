## let 命令

#### 暂时性死区

只要块级作用域内存在 ```let``` 命令，它所声明的变量就 "绑定"（binding）这个区域，不再受外部的影响。

```js
var tmp = 123;

if (true) {
    tmp = "abc";  // tmp is not defined
    let tmp;
}
```

这也就意味着 ```typeof``` 不再是一个百分比安全的操作

```js
typeof x;  // x is not defined

let x;
```

作为比较，如果一个变量根本没有被声明，使用 ```typeof``` 反而不会报错。

```js
typeof y;  // "undefined"
```


#### 比较隐蔽的死区

```js
function bar(x = y, y = 2) {
    return [x, y];
}

bar(); // y is not defined
```

因为参数 ```x``` 默认值等于另一个参数 ```y```，而此时 ```y``` 还没有声明，属于 "死区"，如果 ```y``` 的默认值是 ```x```，就不会报错，因为此时 ```x``` 已经声明了

```js
function bar(x = 2, y = x) {
    return [x, y];
}

bar(); // [2, 2]
```

总之：先声明，在使用




#### 不允许重复声明

```js
function add () {
    let a = 10;  // 报错
    var a = 1;
}

function foo(arg) {
    let arg;  // 报错
}

function func(arg) {
    {
        let arg;  // 不报错
    }
}
```





## 块级作用域

第一种场景，内层变量可能会覆盖外层变量。

```js
var tmp = new Date();

function f() {
    // var tmp
    console.log(tmp);
    if (false) {
        // tmp = "hello world";
        var tmp = "hello world";
    }
}

f();  // undefined
```

第二种场景，用来计数的循环变量泄露为全局变量。

```js
var s = 'hello';

for (var i = 0; i < s.length; i++) {
    console.log(s[i]);
}

console.log(i);  // 5
```

```let``` 实际上为 JavaScript 新增了块级作用域。

```js
function f1() {
    let n = 5;
    if (true) {
        let n = 10;
    }
    console.log(n); // 5（外层代码块不受内层代码块的影响）
}
```

块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。

```js
// IIFE 写法
(function () {
    var tmp = ...;
  ...
} ());

// 块级作用域写法
{
    let tmp = ...;
  ...
}
```


#### 块级作用域与函数声明

ES6 规定，块级作用域之中，函数声明语句的行为类似于 ```let```，在块级作用域之外不可引用

```js
function f() { console.log("I am outside!"); }
(function () {
    if (false) {
        // 重复声明一次函数f
        function f() { console.log("I am inside!"); }
    }

    f();
} ());
```

上面代码在 ES5 中运行，会得到 ```"I am inside!"```，因为在 ```if``` 内声明的函数会被提升到函数头部，实际运行的代码如下：

```js
// ES5 版本
function f() { console.log("I am outside!"); }
(function () {
    function f() { console.log("I am inside!"); }
    if (false) {
    }
    f();
} ());
```

ES6 的运行结果就完全不一样了，会得到 ```"I am outside!"```，因为块级作用域内声明的函数类似 ```let```，对作用域之外没有影响，实际运行的代码如下：

```js
// ES6 版本
function f () { console.log("I am outside!"); }

(function () {
    f();
})
```

很显然，这种行为差异会对老代码产生很大影响。为了减轻因此产生的不兼容问题，ES6 规定，浏览器的实现可以不遵守上面的规定，有自己的行为方式。

* 允许在块级作用域内声明函数

* 函数声明类似于 ```var```，即会提升到全局作用域或函数作用域的头部

* 同时，函数声明还会提升到所在的块级作用域的头部

注意，上面三条规则只对 ES6 的浏览器实现有效，其他环境的实现不用遵守，还是将块级作用域的函数声明当作 ```let``` 处理。

前面那段代码，在 ```Chrome``` 环境下运行会报错。

```js
// ES6的浏览器环境
function f() { console.log("I am outside!"); }
(function () {
    if (false) {
        // 重复声明一次函数f
        function f() { console.log("I am inside!"); }
    }

    f();
} ());
// Uncaught TypeError: f is not a function
```

上面的代码报错，是因为实际运行的是下面的代码。

```js
// ES6的浏览器环境
function f() { console.log("I am outside!"); }
(function () {
    var f;
    if (false) {
        function f() { console.log("I am inside!"); }
    }

    f();
} ());
// Uncaught TypeError: f is not a function
```

考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，也应该写成函数表达式，而不是函数声明语句。

```js
// 函数声明语句
{
    let a = "secret";
    function f() {
        return a;
    }
}

// 函数表达式
{
    let a = "secret";
    let f = function () {
        return a;
    };
}
```

另外，还有一个需要注意的地方。ES6 的块级作用域允许声明函数的规则，只在使用大括号的情况下成立，如果没有使用大括号，就会报错。

```js
// 不报错
"use strict";
if (true) {
    function f() { }
}

// 报错
"use strict";
if (true)
    function f() { }
```




## const 命令

```const``` 声明一个只读常量，一旦声明，常量的值就不能改变

```js
const PI = 3.14;

PI // 3.14

PI = 3;  // Assignment to constant variable.
```

这意味着，```const``` 一旦声明变量，就必须立即初始化，不能留到以后在赋值

```js
const foo;  // Missing initializer in const declaration
```

```const``` 的作用域与 ```let``` 命令相同：只能在声明所在的块级作用域内有效

```js
if (true) {
    const MAX = 5;
}

MAX;  // MAX is not defined
```

总的来说，```const``` 和 ```let``` 有很多相似之处：

* ```const``` 命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用

* ```const``` 声明的常量，也与 ```let``` 一样不可重复声明

需要注意的是：对于复合类型的变量，变量名不指向数据，而是指向数据所在的地址（引用）。```const``` 命令只是保证变量名指向的地址不变，并不保证该地址的数据不变

```js
const foo = {};
foo.prop = 123;

foo.prop
// 123

foo = {};  // Assignment to constant variable.
```

常量 ```foo``` 储存的是一个地址，这个地址指向一个对象。不可变的只是这个地址，即不能把 ```foo``` 指向另一个地址，但对象本身是可变的，所以依然可以为其添加新属性。

```js
const a = [];
a.push('Hello');  // 可执行 数组本身是可写的
a.length = 0;     // 可执行
a = ['Dave'];     // 但是如果将另一个数组赋值给 a，就会报错
```

如果真的想将对象冻结，应该使用 ```Object.freeze``` 方法

```js
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```

将对象彻底冻结的一个函数

```js
var constantize = (obj) => {
    Object.freeze(obj);
    Object.keys(obj).forEach((key, value) => {
        if (typeof obj[key] === 'object') {
            constantize(obj[key]);
        }
    });
};
```


## do 表达式（尚在提案中）

本质上，块级作用域是一个语句，将多个操作封装在一起，没有返回值

```js
{
    let t = f();
    t = t * t + 1;
}
```

但是，在块级作用域之外，没有办法获取到 ```t``` 的值，因为块级作用域不返回值，除非 ```t``` 是全局变量

现在可以在块级作用域之前加上 ```do```，使它变为 ```do``` 表达式

```js
let x = do {
    let t = f();
    t * t + 1; 
}
```

这样一来 ```x``` 就会得到整个块级作用域的返回值