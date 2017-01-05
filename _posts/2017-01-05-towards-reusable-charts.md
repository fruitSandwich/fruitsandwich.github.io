---
layout: post
title:  "制作可复用的图表"
categories: [d3,翻译]
---


Mike Bostock使用d3制作可复用的图表的思路。

原文地址<a href='https://bost.ocks.org/mike/chart/'>https://bost.ocks.org/mike/chart/</a>

我想计划制定一个基于D3封装可复用图表的规则，等着……

```
function chart() {
  // generate chart here
}
```

使用函数；代码复用最标准的形式！！

## 配置

开个玩笑；不是随随便便一个函数就能解决的。实际上我们需要的是可配置的函数，因为大部分的图表需要定制他们的展现和行为。比如说，你可能需要指定宽高或者调色板。配置最简单的方法是给函数传参：

```
function chart(width, height) {
  // generate chart here, using `width` and `height`
}
```
然而这样做对调用者来说显得有些笨重；必须存储图表各种参数，并且在需要更新的时候传递。简单的函数对于高可配的图表来说是不够的。可以用一个配置对象替代，很多图表库都是这么做的：

```
function chart(config) {
  // generate chart here, using `config.width` and `config.height`
}
```
可是，调用者就得管理图表函数（假设你有多种不同的图表类型可选择）还有配置对象。绑定图表配置到图表函数，我们需要一个<a href='http://jibbering.com/faq/notes/closures/'>闭包</a>
```
function chart(config) {
  return function() {
    // generate chart here, using `config.width` and `config.height`
  };
}
```
这样，调用者只需要这样写：
```
var myChart = chart({width: 720, height: 80});
```
随后，调用myChart()更新图表，很简单！


## 更新配置
假如你想要在图表构造之后更新配置呢？或者说你想要获取一个已经存在的图表的配置呢？配置对象被封在闭包里，外部是无法访问的。幸运的是，javascript<a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function'>函数即对象</a>,所以可以把配置属性存到函数本身！
```
var myChart = chart();//myChart是chart()返回的my函数
myChart.width = 720;
myChart.height = 80;
//这里将width和height挂载到my函数本身，所以下面的my函数可以引用到它们
```

chart函数实现稍微改一点点就可以引用挂载在其自身的配置

```
function chart() {
  return function my() {
    // generate chart here, using `my.width` and `my.height`
  };
}
```
用上一点语法糖，可以替换元属性的getter-setter方法就能使用链式调用。这样调用者就可以采用一种更优美的方式来构造表格，chart也可以管理配置参数更改带来的副作用。图表也可以提供默认的配置值。构建一个新图表然后设置两个参数：
```
var myChart = chart().width(720).height(80);
```
更新已经存在的chart也很简单：

```
myChart.height(500);
```
然后是获取值：
```
myChart.height(); // 500
```
chart内部实现稍微改的复杂一些，需要提供getter-setter方法，但是给使用者带来的便利值得开发者额外的努力！（此外，这种模式在你使用一段时间之后会变得非常自然）

```
function chart() {
  var width = 720, // default width
      height = 80; // default height

  function my() {
    // generate chart here, using `width` and `height`
  }

  my.width = function(value) {
    if (!arguments.length) return width;
    width = value;
    return my;
  };

  my.height = function(value) {
    if (!arguments.length) return height;
    height = value;
    return my;
  };

  return my;
}
```

总结：使用getter-setter方法采用闭包的方式实现图表。D3其他可复用对象也使用了相同的模式，包括<a href='https://github.com/d3/d3/wiki/Scales'>scales</a>, <a href='https://github.com/d3/d3/wiki/Layouts'>layouts</a>等等

## 实现
图表现在是可配置的，但是有两个基本成分还没有完成：需要渲染图表到哪个DOM元素（比如特定的Div或者直接就是document.body），还有需要展示的数据。这些在配置中都需要考虑，d3提供了一个更自然地数据和元素的表示：selection。

以selection作为输入，图表会有更大的灵活性。比如可以同事渲染一个图表到多个元素，或者在元素之间移动图表并不需要数据和元素的解绑和重新绑定。可以直接通过配置的变化之间控制图表的更新（比如使用过度而不是瞬时更新）。因此，chart成为渲染数据可视化的橡皮图章。

通过selection调用chart函数最简单的方式是把selection当做参数传递：
```
myChart(selection);
```
同样的，用selection.call也可以：
```
selection.call(myChart);
```
chart函数内部基础实现看起来是这个样子的：
```
function my(selection) {
  selection.each(function(d, i) {
    // generate chart here; `d` is the data and `this` is the element
  });
}
```

## 例子

使这个想法成型，考虑一个简单但非常常见的用例:time-series可视化。time series是随时间变化的变量。可以使用面积图来做可视化，分别将时间和值映射到x轴和y轴对应的位置：

<style>
svg {
  font: 10px sans-serif;
}

.axis path, .axis line {
  fill: none;
  stroke: #000;
  shape-rendering: crispEdges;
}

</style>

<p id='example'></p>
<script src='/asserts/201701/d3.min.js'></script>
<script src='/asserts/201701/time-series-chart.js'></script>
<script>
d3.csv("/asserts/201701/sp500.csv", function(data) {
  var formatDate = d3.time.format("%b %Y");

  d3.select("#example")
    .datum(data)
    .call(timeSeriesChart()
    .x(function(d) { return formatDate.parse(d.date); })
    .y(function(d) { return +d.price; }));
});

</script>


承载这张图表，页面需要一个空的p元素：

```
<p id="example">
```

至于数据，引用外部的CSV文件(<a href='/asserts/201701/sp500.csv'>sp500.csv</a>),头几行是这样子的：

```
date,price
Jan 2000,1394.46
Feb 2000,1366.42
Mar 2000,1498.58
Apr 2000,1452.43
May 2000,1420.6
Jun 2000,1454.6
Jul 2000,1430.83
Aug 2000,1517.68
```

调用timeSeriesChart创建一个新的图表实例，给数据在x（date,日期对象）和y(price,数字)维度上定义访问器：

```
var chart = timeSeriesChart()
    .x(function(d) { return formatDate.parse(d.date); })
    .y(function(d) { return +d.price; });
    var formatDate = d3.time.format("%b %Y");
```

使用一元操作符"+"强制转换price为数字；因为javascript是弱类型语言，所以把加载的数据从字符串转换为数字这是种不错的思路。日期格式需要d3.time.format来解析;"%b"表示月份名的缩写，"%Y"表示四位数字的年份。或许还需要配置其他的选项，这只是个简单的例子使用宽高、合适的间距，其他配置不需要。

最后通过d3.csv加载数据。加载之后，绑定数据到空的p元素，然后使用selection.call来渲染chart!

```
d3.csv("sp500.csv", function(data) {
  d3.select("#example")
      .datum(data)
      .call(chart);
});
```
