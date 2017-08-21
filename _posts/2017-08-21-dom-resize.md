---
layout: post
title:  "dom的onresize事件"
categories: [前端]
---

事件在Web中非常常见。不仅仅是Web,其他带UI的程序中事件作为交互的一种机制应用的也十分广泛。每一个事件都对应着某种操作，比如鼠标的点击，键盘的按下，元素焦点的获取或失去等等。每一个事件都可以绑定一些响应，当事件发生后就会触发他们，比如鼠标点击后就该怎样怎样。


有一些UI组件在初始化的时候必须给一个固定的宽高的容器。不然的话生成的UI就会在一个没有宽高的容器中，也就是说生成的UI没法把给定容器给撑开，比如说EChart。但是给一个固定的宽高比如都是300,那么在大分辨率屏幕下又显的小了。这时候可以给他一个父容器或窗口的比例值，让容器的宽度根据父容器或窗口大小来确定。比如：<a href='/asserts/201708/simpleChart.html'>例子1</a>


现在图表确实是会根据父容器或窗口来确定大小了，但是当窗口大小调整的时候，图表确确不会随着窗口的大小变化而变化体检非常不好。可以改进一下给窗口的onresize事件添加一个响应，当resize窗口后重绘一次图表就行了，或者直接调用ECharts实例的resize方法也行:

```
window.onresize = function () {
     myChart = echarts.init(document.getElementById('chart'));
     myChart.setOption(option);
   }

//或者
window.onresize = myChart.resize;

//或者使用事件绑定
window.addEventListener('resize',myChart.resize);
```

结果如下，可以看到这个图表可以随着窗口大小变化了:<a href='/asserts/201708/responseChart.html'>例子2</a>

但是如果想要图表随着它的父容器大小而变化怎么办呢?比如下面图表的容器大小是可以拖动的(Chorme、Firefox下)，想要让里面的图表也随着改变大小。
<script src="../asserts/echarts.simple.min.js"></script>
<div id="parent" style="border:2px solid;padding:10px 40px;width:100%;resize:both;overflow:auto;">
  <div id="chart" style="width: 100%;height: 100%;">
  </div>
</div>
<script>
  var option = {
    title: {
      text: 'ECharts 入门示例'
    },
    tooltip: {},
    legend: {
      data: ['销量']
    },
    xAxis: {
      data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
    },
    yAxis: {},
    series: [{
      name: '销量',
      type: 'bar',
      data: [5, 20, 36, 10, 10, 20]
    }]
  };
  var myChart = echarts.init(document.getElementById('chart'));
  myChart.setOption(option);
  var parentElement = document.getElementById('parent');
  parentElement.addEventListener('resize', myChart.resize)
</script>

<b>父容器绑定的onresize事件并没有生效。</b>

查了一下onresize API,onresize 一般只用在window对象或者body对象上,普通dom不一定生效，曾经有人给webkit提过bug：<a href='https://bugs.webkit.org/show_bug.cgi?id=17969'>https://bugs.webkit.org/show_bug.cgi?id=17969</a>。也有人测过这个事件:<a href='https://www.quirksmode.org/dom/events/resize.html'>https://www.quirksmode.org/dom/events/resize.html</a>
<img src='/asserts/201708/onresize.png'>

有一些版本的IE上普通dom的onresize事件是生效的,我试了一下IE10和IE9可以成功触发但IE11不行。这就很尴尬了如果想要监听某个普通dom的大小变化怎么办呢。

我想到了轮询脏检查，然后试了一下：

```
function domResize(dom, callback) {
  if (!dom)
    return;
  var lastWidth = dom.clientWidth;
  var lastHeight = dom.clientHeight;

  setInterval(function () {
    if (lastWidth === dom.clientWidth && lastHeight === dom.clientHeight)
      return;
    if ((typeof callback) === 'function') {
      callback({width: lastWidth, height: lastHeight},
        {width: dom.clientWidth, height: dom.clientHeight});
      lastWidth = dom.clientWidth;
      lastHeight = dom.clientHeight;
    }
  }, 100);
}
var parentElement = document.getElementById('parent');
domResize(parentElement, myChart.resize)
```

<div id="parent2" style="border:2px solid;padding:10px 40px;width:100%;resize:both;overflow:auto;">
  <div id="chart2" style="width: 100%;height: 100%;">
  </div>
</div>
<script>
  function domResize(dom, callback) {
    if (!dom)
      return;
    var lastWidth = dom.clientWidth;
    var lastHeight = dom.clientHeight;

    setInterval(function () {
      if (lastWidth === dom.clientWidth && lastHeight === dom.clientHeight)
        return;
      if ((typeof callback) === 'function') {
        callback({width: lastWidth, height: lastHeight},
          {width: dom.clientWidth, height: dom.clientHeight});
        lastWidth = dom.clientWidth;
        lastHeight = dom.clientHeight;
      }
    }, 100);
  }
  var myChart2 = echarts.init(document.getElementById('chart2'));
  myChart2.setOption(option);
  var parentElement2 = document.getElementById('parent2');
  domResize(parentElement2, myChart2.resize)
</script>

可以触发dom的更改，挺好，然而还有个解决方案:<a href='http://www.backalleycoder.com/2013/03/18/cross-browser-event-based-element-resize-detection/'>http://www.backalleycoder.com/2013/03/18/cross-browser-event-based-element-resize-detection/</a>

 <script src="../asserts/201708/resize.js"></script>
<div id="parent3" style="border:2px solid;padding:10px 40px;width:300px;resize:both;overflow:auto;">
  <div id="chart3" style="width: 100%;height: 100%;">
  </div>
</div>
<script>
  var myChart3 = echarts.init(document.getElementById('chart3'));
  myChart3.setOption(option);
  var parentElement3 = document.getElementById('parent3');
  addResizeListener(parentElement3, myChart3.resize);
</script>
