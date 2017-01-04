---
layout: post
title:  "三个小圆圈"
categories: [d3,翻译]
---

从前有三个小圆圈

<svg width="720" height="120">
  <circle cx="40" cy="60" r="10"></circle>
  <circle cx="80" cy="60" r="10"></circle>
  <circle cx="120" cy="60" r="10"></circle>
</svg>

```
<svg width="720" height="120">
  <circle cx="40" cy="60" r="10"></circle>
  <circle cx="80" cy="60" r="10"></circle>
  <circle cx="120" cy="60" r="10"></circle>
</svg>
```

这份教程将向你展示怎样用选择器来控制它们

<!--more-->

<script src='/asserts/201701/d3.min.js'></script>

## 选择元素
<a href='https://github.com/d3/d3/wiki/Selections#selectAll'>d3.selectAll</a>方法需要一个<a href='https://www.w3.org/TR/CSS2/selector.html'>选择器</a>,例如"circle",然后返回一个匹配元素的集合<i>selection</i>:
```
var circle = d3.selectAll("circle");
```
通过selection我们可以对选中元素做各种更改。比如，我们可以使用 selection.style改变填充颜色，使用selection.attr来添加半径：

```
circle.style("fill", "steelblue");
circle.attr("r", 30);
```

上面的代码为所有选中的元素设置了相同的样式和属性。

<svg width="720" height="120">
  <circle cx="40" cy="60" r="30" style="fill:steelblue;"></circle>
  <circle cx="80" cy="60" r="30" style="fill:steelblue;"></circle>
  <circle cx="120" cy="60" r="30" style="fill:steelblue;"></circle>
</svg>

```
<svg width="720" height="120">
  <circle cx="40" cy="60" r="30" style="fill:steelblue;"></circle>
  <circle cx="80" cy="60" r="30" style="fill:steelblue;"></circle>
  <circle cx="120" cy="60" r="30" style="fill:steelblue;"></circle>
</svg>
```
我们也可以在每一个元素的基础上通过匿名函数来赋值。这个函数为每一个选中元素赋一次值。使用匿名函数来技术属性值在d3中非常广泛，尤其是<a href='https://github.com/d3/d3/wiki/Quantitative-Scales'>比例尺</a>和<a href='https://github.com/d3/d3/wiki/SVG-Shapes'>形状</a>。给每个圆圈的x坐标赋一个随机值：
```
circle.attr("cx", function() { return Math.random() * 720; });
```
如果反复运行这段代码，圆圈们就会开始跳起来:

<svg id="circle-dance" width="720" height="120">
  <circle cx="317.8811214861489" cy="60" r="30" style="fill:steelblue;"></circle>
  <circle cx="592.7874876094867" cy="60" r="30" style="fill:steelblue;"></circle>
  <circle cx="624.8980020519111" cy="60" r="30" style="fill:steelblue;"></circle>
</svg>

<script>(function() {

function dance() {
  var circle = d3.selectAll("#circle-dance circle"),
      span = d3.selectAll(".circle-dance-x"),
      data = d3.range(3).map(function() { return Math.random() * 720; });

  circle.data(data).attr("cx", function(d) { return d; });
  span.data(data).text(function(d) { return d; });
}

dance();
setInterval(dance, 2500);

})();</script>


## 绑定数据
更通常的情况是，我们使用<i>数据</i>来驱动圆圈的展现。如果说我们想要让这些圆圈展示三个数：32，57，112。<a href='https://github.com/d3/d3/wiki/Selections#data'>selection.data</a>方法可以绑定数字到圆圈:
```
circle.data([32, 57, 112]);
```
数据被指定为一个值数组；数组映射到被选元素的集合，也是一个数组。上面的代码中，第一个数字(32)被绑定到第一个圆圈（按Dom中定义顺序的第一个元素），第二个数字绑定到第二个圆圈，等等。
数据绑定完之后，可作为给属性或样式赋值的匿名函数中第一个参数使用。按习惯，我们通常使用d作为参数名给绑定值命名。使用数据来给半径赋值：
```
circle.attr("r", function(d) { return Math.sqrt(d); });
```
结果是这样的：
<svg width="720" height="120">
  <circle cx="40" cy="60" r="5.656854249492381" style="fill:steelblue;"></circle>
  <circle cx="80" cy="60" r="7.54983443527075" style="fill:steelblue;"></circle>
  <circle cx="120" cy="60" r="10.583005244258363" style="fill:steelblue;"></circle>
</svg>

```
<svg width="720" height="120">
  <circle cx="40" cy="60" r="5.656854249492381" style="fill:steelblue;"></circle>
  <circle cx="80" cy="60" r="7.54983443527075" style="fill:steelblue;"></circle>
  <circle cx="120" cy="60" r="10.583005244258363" style="fill:steelblue;"></circle>
</svg>
```
请注意，SVG中坐标轴的原点在左上角。

## Entering 元素
如果我们有四个数字要展示，而不是三个呢？我们没有足够的圆圈，我们需要构造更多的元素来展示我们的数据。你可以手工添加新的节点，但更好的方法是使用数据绑定计算的enter选择器。
当绑定数据到元素的时候，d3把剩余的数据（或者说相对应的“消失”的元素）放到enter selection。在只有三个圆圈的情况下，第四个数将会被放到enter selection，而另外三个会被直接通过selection.data返回（在update selection中）

通过添加到enter selection，我们可以为多余的数字创建新的圆圈。新的圆圈会被添加到父元素中。所以，我们可以先select“svg”元素，然后select all “circle”元素，最后给他们绑定数据。
```
var svg = d3.select("svg");

var circle = svg.selectAll("circle")
    .data([32, 57, 112, 293]);

var circleEnter = circle.enter().append("circle");
```
entering 元素已经绑定了元素，所以我们可以使用数据来计算属性和样式，当然也可以设置常数属性：
```
circleEnter.attr("cy", 60);
circleEnter.attr("cx", function(d, i) { return i * 100 + 30; });
circleEnter.attr("r", function(d) { return Math.sqrt(d); });
```
这样我们就有了四个圆圈

<svg width="720" height="120">
  <circle cx="30" cy="60" r="5.656854249492381" style="fill:steelblue;"></circle>
  <circle cx="130" cy="60" r="7.54983443527075" style="fill:steelblue;"></circle>
  <circle cx="230" cy="60" r="10.583005244258363" style="fill:steelblue;"></circle>
  <circle cx="330" cy="60" r="17.11724276862369" style="fill:steelblue;"></circle>
</svg>

```
<svg width="720" height="120">
  <circle cx="30" cy="60" r="5.656854249492381" style="fill:steelblue;"></circle>
  <circle cx="130" cy="60" r="7.54983443527075" style="fill:steelblue;"></circle>
  <circle cx="230" cy="60" r="10.583005244258363" style="fill:steelblue;"></circle>
  <circle cx="330" cy="60" r="17.11724276862369" style="fill:steelblue;"></circle>
</svg>
```

再考虑一下极限情况，假如一个元素都没有呢，例如一个空页面？然后我们把数据绑定到一个空选择集，所有的数据都成为enter。

这种模式很常见，你将会经常看到selectAll+data+enter+append方法被频繁调用，一个接一个地级联调用。尽管很常见，记住这仅仅是数据绑定的一个特殊例子：
```
svg.selectAll("circle")
    .data([32, 57, 112, 293])
  .enter().append("circle")
    .attr("cy", 60)
    .attr("cx", function(d, i) { return i * 100 + 30; })
    .attr("r", function(d) { return Math.sqrt(d); });
```

这种enter模式经常结合方法的<a href="https://en.wikipedia.org/wiki/Method_chaining">链式调用</a>，缩写代码的另一种方式。d3方法返回他们更改后的选择集，你可以通过链式调用应用多次操作到同一个选择集。

## Exiting 元素
你经常有和enter相反的问题：你有过多的已存在的元素，你想要删掉一些。当然，你可以选择元素然后手动删掉它们，但是通过数据绑定计算exit selection更高效。
exit selection是和enter slection相反的：它包含了多余的元素，这些元素没有对应的绑定数据。
```
var circle = svg.selectAll("circle")
    .data([32, 57]);
```

剩下的要做的，就是删除exitting元素
```
circle.exit().remove();
```
然后我们就只有两个圆圈了：

<svg width="720" height="120">
  <circle cx="30" cy="60" r="5.656854249492381" style="fill:steelblue;"></circle>
  <circle cx="130" cy="60" r="7.54983443527075" style="fill:steelblue;"></circle>
</svg>

```
<svg width="720" height="120">
  <circle cx="30" cy="60" r="5.656854249492381" style="fill:steelblue;"></circle>
  <circle cx="130" cy="60" r="7.54983443527075" style="fill:steelblue;"></circle>
</svg>
```
当然，你不必须立即删除exiting元素；例如可以应用一个过渡让他们淡出或滑出。

## 总结

放到一块，考虑数据绑定元素所有可能的结果：
1. enter-进入的元素，entering阶段
2. update-当前元素，staying阶段
3. exit-移出的元素，exiting阶段

通常，数据通过索引绑定：第一个元素绑定第一个数据。因此，enter或exit集合就会为空，或者两个都为空。如果数据比元素多，多余的数据就会在enter selection。如果数据比元素少，多余的元素就会在exit selection.

你可以精确的控制哪个数据绑定到哪个元素，通过给selection.data指定一个key function。例如通过使用identity 函数，可以重新给圆圈绑定新数据.

```
var circle = svg.selectAll("circle")
    .data([32, 57, 293], function(d) { return d; });

circle.enter().append("circle")
    .attr("cy", 60)
    .attr("cx", function(d, i) { return i * 100 + 30; })
    .attr("r", function(d) { return Math.sqrt(d); });

circle.exit().remove();
```
