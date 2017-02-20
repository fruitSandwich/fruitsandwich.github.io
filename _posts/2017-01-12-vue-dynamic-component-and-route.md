---
layout: post
title:  "vue动态组件和路由"
categories: [javascript,vue]
---

在软件开发领域组件化是一个非常实用的思想，核心概念是内聚性、低耦合。

意思就是：

哥，我想按时回家哄妹子！！！你怎么写代码我不管，你的功能全在这你这儿实现（内聚性），不要让我还帮写你那块功能。另外，哥，求你了，你代码不要影响我的代码（低耦合性）。

下面要说的是Vue组件以及如何动态添加组件、路由组件。

## 一、Vue、Vue组件
Vue.js（读音 /vjuː/, 类似于 view） 是一套构建用户界面的 渐进式框架。与其他重量级框架不同的是，Vue 采用自底向上增量开发的设计。Vue 的核心库只关注视图层，并且非常容易学习，非常容易与其它库或已有项目整合。-----<a href='http://cn.vuejs.org/'>Vue中文官网</a>

Vue的渐进式、自底向上增量开发理念hit到了我，尤大的文章：<a href='http://mp.weixin.qq.com/s?__biz=MzIwNjQwMzUwMQ==&mid=2247484393&idx=1&sn=142b8e37dfc94de07be211607e468030&chksm=9723612ba054e83db6622a891287af119bb63708f1b7a09aed9149d846c9428ad5abbb822294&mpshare=1&scene=1&srcid=1026oUz3521V74ua0uwTcIWa&from=groupmessage&isappinstalled=0#wechat_redirect&utm_source=tuicool&utm_medium=referral'>Vue作者尤雨溪：Vue 2.0，渐进式前端解决方案</a>


如果使用构建工具，Vue提供了一种扩展名为.vue的单文件组件，这样一个.vue文件就是一个组件，可以使用项目目录规整组件结构。
![image](/asserts/201701/vue-component.png)

构建复杂页面可以导入组件进行组装，比如：
![image](/asserts/201701/vue-page.png)

使用Vue组件方便了构建复杂页面，也可以根据数据渲染组件内容。但是想让页面使用哪些组件也通过数据来决定呢？我的目标是在页面上可以手动操作决定页面使用什么组件。

## 二、动态组件
首先需要把有什么组件管理起来,把所有用到的组件注册到一个对象中：

```
import A from './components/A.vue'
import B from './components/B.vue'
import Hello from './components/Hello.vue'
let component = {
  a: A,
  b: B,
  hello: Hello
};
```

定义一个全局Vue组件dynamicComponent，让它根据传入的组件名渲染出对应组件：

```
Vue.component('dynamicComponent', {
  props: ['name'],
  render: function (h) {
    return createElement(components[this.name], {})
  },
});
```

使用dynamicComponent加载组件：

```
<dynamic-component name="a"></dynamic-component>
<dynamic-component name="b"></dynamic-component>
```

还可以根据数据来确定加载什么组件，彻底做到动态性：

```
<dynamic-component :name="componentName"></dynamic-component>
```

在线例子：<a href='/asserts/201701/vue-dynamic/index.html#/component'>查看</a>

![image](/asserts/201701/vue-dynamic/vue-component-result.png)

效果：选择组件添加后会在页面上动态显示出相应组件的内容。

## 三、动态路由

路由"route"是指根据url分配到对应的处理程序。在web单页应用程序中，一般控制页面视图的切换。

下面是vue-router库中路由的定义方法。

```
// 1. 定义（路由）组件。
// 可以从其他文件 import 进来
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]
```

可以看到，路由由两部分信息构成：
- path:url相关路径
- component:路由对应的组件

官方文档使用了const常量来定义routes，十分清晰的表明定义好路由后不要改。

但是我就是想改。。。。![image](/asserts/201701/bz-face.png)

怎么改？页面加载完之后路由已经确定了。选择在初始化的时候做文章。路由初始化之前把路由信息查出来，通过动态的路由数据构造路由表，数据一般是从服务器来的，所以使用了<a href='http://fruitsandwich.github.io/promise-summary/'>异步模式</a>

```
const routes = []
function initRoutes(dynamicRoutes) {
  dynamicRoutes.forEach(item => {
    try {
      routes.push({path: '/' + item.path, component: component[item.name]})
      //通过动态数据的path和componentName构造路由表
    } catch (e) {
      console.error(`component message:${JSON.stringify(item)},error:${e.message}`)
    }
  })
}
import routerDao from './RouterDao'
//异步路由数据的Promise模块
routerDao.list()
  .then(initRoutes)
  .then(function () {
    //构造路由
    var router = new VueRouter({
      routes
    });
    //渲染View
    new Vue({
      router,
    }).$mount('#app')
  })

```

在线例子：<a href='/asserts/201701/vue-dynamic/index.html#/'>查看</a>
![image](/asserts/201701/vue-dynamic/vue-page-result.png)


效果：添加后，刷新页面，点击链接跳转到相应路由
> 注意：添加路由后，需要刷新页面重新初始化路由才能生效。
