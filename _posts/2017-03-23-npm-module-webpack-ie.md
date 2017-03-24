---
layout: post
title:  "npm 模块 Webpack打包 IE下的问题"
categories: [npm]
---

前面提到了可以<a href='/npm-repository-server'>安装私有的npm仓库</a>，通过内网发布和安装js模块。还有一种方法可以在内网环境使用npm安装js模块，那就是npm install git+：

```
npm install git+https://github.com/jquery/jquery.git
```

这样可以把git远程仓库上托管的模块通过npm下载到本地并通过npm进行管理。那么只要搭建一个内网git仓库就可以结合npm管理主项目和其子项目,不过git仓库管理的项目必须符合npm规范。

package.json中需要指定模块出口,比如jquery:

```
{
  "name": "jquery",
  "title": "jQuery",
  "description": "JavaScript library for DOM operations",
  "version": "3.2.2-pre",
  "main": "dist/jquery.js",//模块出口
  "homepage": "https://jquery.com",
}
```

上面的jquery主出口是经过编译打包好的单文件模块，其实只要符合模块化规范，在Webpack等模块化打包工具，主出口也可以是引用其他模块后合并输出，比如：

src/export.js:
```
//CMD规范
const js1 = require('js1');

function b() {
    js1.a();
    console.log('b')
}

module.exports = {b};
```

package.json:

```
{
   "main": "src/export.js",
}
```

主项目通过安装好上面这个模块后，会同时安装js1模块，主项目引用子模块：

```
const js2 = require('js2');
js2.b()
```
CMD规范下，Node环境可直接运行得到正确结果。

通过WebPack打包后在浏览器运行，Chrome、Firefox等可得到正确结果，IE下报错：

```
SCRIPT1003: 缺少 ':'
```

![image](/asserts/201703/npm-module-ie.png)

找了半天，发现原因在于,导出模块时使用的对象字面量使用了<a href='http://es6.ruanyifeng.com/#docs/object'>ES6对象的简洁表示法</a>。然而Webpack打包转码可能只转码了项目自身的代码，而引入的子模块却直接通过eval直接运行了。所以才导致了Chrome、Firefox等支持ES6的浏览器没有问题，IE才报错，而且通过eval执行的代码好像加polyfill也没管用···

所以把export.js中ES6写法改一下：

```
const js1 = require('js1');

function b() {
    js1.a();
    console.log('b')
}

module.exports = {b：b};
```

这样就可以级联子项目一块打包编译在IE下也能支持。
