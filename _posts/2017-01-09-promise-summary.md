---
layout: post
title:  "javascript异步编程和Promise"
categories: [javascript]
---

javascript是单线程的，但有两种执行模式:同步和异步。

关于同步和异步的效果我写了一个<a href='http://fruitsandwich.github.io/webTraining/lession03/asynchronous.html'>示例</a>。

同步就是执行顺序和任务顺序时一致，同步的；异步任务都需要指定一个或多个回调函数，前一个任务执行结束后，不是执行后一个任务而是执行回调函数，后一个任务不等前一个任务执行完就执行，程序的执行顺序和任务排列顺序是不一致的，异步的。

关于异步编程模式阮一峰总结了<a href='http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html'>4个方法</a>

1. 回调函数
2. 事件监听
3. 发布订阅
4. **Promise对象**

## 一、Promise

这里来说说<a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise'>Promise</a>。
MDN关于Promise中文给出的解释是：

```
所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。
```

ES6将Promise写进了语言标准统一了用法。比较详细的API:<a href='http://es6.ruanyifeng.com/#docs/promise'>ECMAScript6入门</a>

Promise的构造方法：

```
var promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

可以看到Promise接收一个函数作为参数，函数有两个参数，一个是表示操作成功的resolve函数，一个是表示操作失败调用的reject函数。

构造的时候不用关心resolve和reject函数，因为它们由javascript引擎提供，不用自己部署。那么啥时候关心或者说啥时候指定成功和失败函数呢。Promise实例生成后使用实例的then方法指定成功和失败的回调函数。

```
var resolve = function (value) {
  //异步成功后的操作
}
var reject = function (error) {
  //异步失败后的操作
}

promise.then(resolve,reject);
```

完整的例子：

```
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});
```

## 二、Promise与Ajax

光看API还有简单例子看不出Promise的强大，下面通Promise来改写最常见的Ajax。为了简化代码，采用jquery的写法：

```
var ajaxPromise = function (url) {
  return new Promise(function (resolve, reject) {
    $.ajax({
      url: url,
      success: resolve,
      error: reject
    })
  })
}
ajaxPromise('./a.json')//执行异步获取
  .then(function (data) {//异步结束调用
    console.log(data)
  })
```

上面使用Promise封装了jquery的ajax方法，把通过回调函数的方式改写成Promise方式，使得代码执行的顺序看起来没有回调那么拧巴。一次ajax调用除了代码顺序和执行顺序保持了一致以外看不出来Promise有什么好处。但是当异步操作次数多而且调用结果需要关联的时候就很需要了。

比如需要ajax获取两个资源a.json和b.json，最后对两个资源合并处理。先来看一段代码：

```
var arr = [];
$.get('./a.json', function (data) {
  arr.push(data)
})
$.get('./b.json', function (data) {
  arr.push(data)
})
console.log(arr)
```

写出上面这种代码的是没有理解javascript异步，后面使用arr的时候前面的异步调用还没有执行完，这时候arr是没有内容的。想要使用异步调用的结果，可以把ajax调用改成同步但这样页面就会卡死（参见最开始的例子），也可以使用回调嵌套：

```
var arr = [];
$.get('./a.json', function (data) {
  arr.push(data)
  $.get('./b.json', function (data) {
    arr.push(data)
    console.log(arr)
  })
})
```

使用回调嵌套可以得到正确的结果，但是看起来就没那么好看了，回调一多就成了<a href='http://callbackhell.com/'>噩梦</a>（最后的闭合标签和容易错，比如有人写过<a href='http://blog.jobbole.com/37863/'>21个嵌套的回调</a>）。如果采用Promise封装的ajax，可以很优雅的解决它：

```
var arr = [];
ajaxPromise('./a.json')
  .then(function (data) {
    arr.push(data)
    return ajaxPromise('./b.json')
  })
  .then(function (data) {
    arr.push(data)
    console.log(arr)
  });
```

promise实例的then方法返回值是一个新的promise实例，可以采用链式写法继续调用then方法，直到得到最后结果。

如果比两个还要多的ajax调用的话，还可以使用Promise.all()方法将多个promise实例包装成一个promise实例，而且每个promise的返回值在合成的promise实例中将组成一个数组。

合并promise使代码更加优雅：

```
var urls = ['./a.json', './b.json'];
var promises = urls.map(function (url) {
  return ajaxPromise(url)
});
Promise.all(promises).then(function (values) {
  console.log(values);
})
```
