---
title: 开发工具-Babel扫盲
date: 2018-05-22 21:30:43
tags: [开发工具, 构建工具]
categories: 基础储备
description: Babel 扫盲
---
<!-- more -->

## Babel 前言

#### 参考资料

- [Babel 入门教程](http://www.ruanyifeng.com/blog/2016/01/babel.html)

#### 为什么要用 Babel?

近年来，JS 发展迅速，尤其 ES6 之后，许多新特性使得 JS 越来越规范化，也使开发者如鱼得水，但由于浏览器厂商对新特性的支持程度不一，对于有兼容性要求的应用让我们如鲠在喉。Babel 的出现便是解决这种困境。“Babel is a JavaScript compiler，Use next generation JavaScript, today.”

#### Babel 的特性？

- Babel 能够转换新的语法，无需等待浏览器支持。如下：
```
Arrow functions
Async functions
Async generator functions
Block scoping
Block scoped functions
Classes
Class properties
Computed property names
Constants
Decorators
Default parameters
Destructuring
Do expressions
Exponentiation operator
For-of
Function bind
Generators
Modules
Module export extensions
New literals
Object rest/spread
Property method assignment
Property name shorthand
Rest parameters
Spread
Sticky regex
Template literals
Trailing function commas
Type annotations
Unicode regex
```

- 由于 Babel 只转换新语法，对于一些新的对象，原生方法我们则需要使用 polyfill（`babel-polyfill`），使用它时需要在你应用程序的入口点顶部或打包配置中引入
```
ArrayBuffer
Array.from
Array.of
Array#copyWithin
Array#fill
Array#find
Array#findIndex
Function#name
Map
Math.acosh
Math.hypot
Math.imul
Number.isNaN
Number.isInteger
Object.assign
Object.getOwnPropertyDescriptors
Object.is
Object.entries
Object.values
Object.setPrototypeOf
Promise
Reflect
RegExp#flags
Set
String#codePointAt
String#endsWith
String.fromCodePoint
String#includes
String.raw
String#repeat
String#startsWith
String#padStart
String#padEnd
Symbol
WeakMap
WeakSet
```
- Babel 能够转换 JSX 语法并去除类型注释（`babel-preset-react`）
- 可以配置插件，可以使用自己的插件和已有插件去转换代码。
- 可调试，支持 Source Map 可以轻松调试编译后代码

## 使用 Babel

#### Babel 的一些模块以及使用

- .babelrc
  + Babel 最常用的使用方式就是 .babelrc，该文件用来设置转码规则和插件
  + .babelrc，存放在项目的根目录下。使用Babel的第一步，就是配置这个文件
  + 也可以在 package.json 中如下指定 .babelrc 的配置
  + Babel 会在正在被转录的文件的当前目录中查找一个 .babelrc 文件。 如果不存在，它会遍历目录树，直到找到一个 .babelrc 文件，或一个 package.json 文件中有 "babel": {}

```JSON
{
  "name": "my-package",
  "version": "1.0.0",
  "babel": {
    // my babel config here
  }
}
```

env 选项 — 在特定环境的时候，您可以用 env 选项来设置特定的配置

```JSON
// 特定情景的设置项会被合并、覆盖到没有特定环境的设置项中
// env 选项的值将从 process.env.BABEL_ENV 获取，如果没有的话，则获取 process.env.NODE_ENV 的值，它也无法获取时会设置为 "development"

{
  "env": {
    "production": {
      "plugins": ["transform-react-constant-elements"]
    }
  }
}
```


>注意：以下所有Babel工具和模块的使用，都必须先写好 .babelrc

- babel-core 核心依赖包
  + 如果某些代码需要调用 Babel 的 API 进行转码，就要使用 babel-core 模块

```
$ npm install babel-core --save

var babel = require('babel-core');

// 字符串转码
babel.transform('code();', options);
// => { code, map, ast }

// 文件转码（异步）
babel.transformFile('filename.js', options, function(err, result) {
  result; // => { code, map, ast }
});

// 文件转码（同步）
babel.transformFileSync('filename.js', options);
// => { code, map, ast }

// Babel AST转码
babel.transformFromAst(ast, code, options);
// => { code, map, ast }
```

- babel-polyfill
  + Babel 默认只转换新的 JavaScript 句法（syntax），而不转换新的 API，比如 Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，以及一些定义在全局对象上的方法（比如 Object.assign）都不会转码。如果想让这个方法运行，必须使用babel-polyfill，为当前环境提供一个垫片

```
$ npm install --save babel-polyfill

import 'babel-polyfill';
// 或者
require('babel-polyfill');
// 在 webpack 中
module.exports = {
  entry: ["babel-polyfill", "./app/js"]
};
//在浏览器中
<script src=".....dist/polyfill.js " />
```

- 命令行转码 babel-cli

```
$ npm install --global babel-cli

# 转码结果输出到标准输出
$ babel example.js

# 转码结果写入一个文件
# --out-file 或 -o 参数指定输出文件
$ babel example.js --out-file compiled.js
# 或者
$ babel example.js -o compiled.js

# 整个目录转码
# --out-dir 或 -d 参数指定输出目录
$ babel src --out-dir lib
# 或者
$ babel src -d lib

# -s 参数生成 source map 文件
$ babel src -d lib -s
```

- babel-node
  + babel-cli 工具自带一个 babel-node 命令，提供一个支持 ES6 的 REPL 环境，它支持Node的REPL环境的所有功能，而且可以直接运行ES6代码。
  + 它不用单独安装，而是随 babel-cli 一起安装。然后，执行 babel-node 就进入 PEPL 环境。

```
$ babel-node
> (x => x * 2)(1)
2

// babel-node命令可以直接运行ES6脚本
$ babel-node es6.js
2
```

- babel-register
  + babel-register 模块改写 require 命令，为它加上一个钩子。此后，每当使用 require 加载.js、.jsx、.es和.es6后缀名的文件，就会先用Babel进行转码。
  + 只会对 require 命令加载的文件转码，而不会对当前文件转码
  + 它是实时转码，所以只适合在开发环境使用。

```
$ npm install --save-dev babel-register

// 然后，就不需要手动对index.js转码了
require("babel-register");
require("./index.js");
```

