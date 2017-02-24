---
layout: post
title:  "javascript模块化"
categories: [javascript,npm]
---

## 1.javascript模块化那些事
从前从前，javascript是没有模块化概念的，无法将一个大程序拆分成互相依赖的小文件，再用简单的方法拼装起来。但是其他语言都有这项功能，比如java的import、 Ruby 的require、Python 的import，甚至就连 CSS 都有@import，但是 JavaScript 任何这方面的支持都没有，这对开发大型的、复杂的、多人协作的项目形成了巨大障碍。

历史上出现了各种包装js代码形成模块的解决方案和规范，其中最著名的是CommonJS规范，以及后来演化出来的AMD与CMD以及他们对应的RequireJS和SeaJS库。

这些解决方案和模块化规范的演化历史可以看看下面几篇文章

- <a href='http://web.jobbole.com/83761/'>JavaScript 模块化历程</a>
- <a href='https://github.com/seajs/seajs/issues/588'>前端模块化开发那点历史</a>

## 2.CommonJS与nodejs
CommonJS 是以在浏览器环境之外构建 JavaScript 生态系统为目标而产生的项目，比如在服务器和桌面环境中。

这个项目最开始是由 Mozilla 的工程师 Kevin Dangoor 在2009年1月创建的，当时的名字是 ServerJS。

- 我在这里描述的并不是一个技术问题，而是一件重大的事情，让大家走到一起来做决定，迈出第一步，来建立一个更大更酷的东西。 —— Kevin Dangoor's <a href='http://www.blueskyonmars.com/2009/01/29/what-server-side-javascript-needs/'>What Server Side JavaScript needs</a>

CommonJS 规范是为了解决 JavaScript 的作用域问题而定义的模块形式，可以使每个模块它自身的命名空间中执行。该规范的主要内容是，模块必须通过 module.exports 导出对外的变量或接口，通过 require() 来导入其他模块的输出到当前模块作用域中。

```
// example.js
var x = 5;
var addX = function (value) {
  return value + x;
};
module.exports.x = x;
module.exports.addX = addX;
```

上面代码通过module.exports输出变量x和函数addX。

require方法用于加载模块。

```
var example = require('./example.js');

console.log(example.x); // 5
console.log(example.addX(1)); // 6
```

更详细的CommonJS以及NodeJS模块相关的资料：

- <a href='http://www.commonjs.org/specs/modules/1.0/'>CommonJS Modules</a>
- <a href='http://nodejs.cn/api/modules.html#modules_addenda_package_manager_tips'>NodeJS module</a>
- <a href='http://webpackdoc.com/commonjs.html'>CommonJS 规范</a>
- <a href='http://javascript.ruanyifeng.com/nodejs/module.html'>《JavaScript 标准参考教程（alpha）》，by 阮一峰：CommonJS规范</a>

## 3.AMD与RequireJS

AMD（异步模块定义）是为浏览器环境设计的，因为 CommonJS 模块系统是同步加载的，当前浏览器环境还没有准备好同步加载模块的条件。

- CommonJS其实也有浏览器端的实现，其原理是现将所有模块都定义好并通过 id 索引，这样就可以方便的在浏览器环境中解析了，可以参考 <a href='https://github.com/Stuk/require1k'>require1k</a> 和 <a href='https://github.com/ruanyf/tiny-browser-require'>tiny-browser-require</a> 的源码来理解其解析（resolve）的过程。

AMD 定义了一套 JavaScript 模块依赖异步加载标准，来解决同步加载的问题。

模块通过 define 函数定义在闭包中，格式如下：

```
define(id?: String, dependencies?: String[], factory: Function|Object);
```

1. id：指定义中模块的名字，可选；如果没有提供该参数，模块的名字应该默认为模块加载器请求的指定脚本的名字。

2. 依赖dependencies：是一个当前模块依赖的，已被模块定义的模块标识的数组字面量。
依赖参数是可选的，如果忽略此参数，它应该默认为["require", "exports", "module"]。
```
define(function(require, exports, module) {}）
```

3. 工厂方法factory，模块初始化要执行的函数或对象。如果为函数，它应该只被执行一次。如果是对象，此对象应该为模块的输出值。

比如：
```
define('myModule', ['jquery'], function($) {
    // $ 是 jquery 模块的输出
    $('body').text('hello world');
});
// 使用
define(['myModule'], function(myModule) {});
```
