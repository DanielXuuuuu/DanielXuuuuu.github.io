---
title: CSS布局学习 | 圣杯、双飞翼布局
date: 2019-11-24 10:28:46
categories: 
	- CSS学习笔记
tags: 
	- CSS
	- 前端
---

> CSS学习笔记
>
> 本篇笔记主要记录了经典的圣杯、双飞翼布局的实现方式

<!-- more -->

圣杯和双飞翼布局是最为经典和基础的两个布局方式了，两者的目的都是为了实现一个三栏的布局，其共同点如下：

+ 两侧的宽度固定，中间的宽度能够自适应调整；
+ 中间部分在DOM结构上优先，以便先行渲染
+ 允许任意列的高度最高

### 圣杯布局

#### DOM结构

```html
<body>
    <div id="header">This is header</div>
    <div id="container">
        <div id="center" class="column">center</div>
        <div id="left" class="column">left</div>
        <div id="right" class="column">right</div>
    </div>
    <div id="footer">This is footer</div>
</body>
```

基本的DOM结构如上，主体的三列被包括在`container`中，注意`center`在最前面。

#### CSS布局

我们首先给设定如下布局，先给每一栏设定不同的高度和背景颜色：

```css
*{
    padding: 0;
    margin: 0;
}

#header, #footer{
    background-color: gray;
    padding: 5px;
    text-align: center;
}

#center{
    background-color: blueviolet;
    height: 250px;
}

#left{
    background-color: bisque;
    height: 290px;
}

#right{
    background-color: aqua;
    height: 300px;
}
```

得到以下结果：

{% asset_img 1.png %}

然后增加如下代码：

```css
/*为左右两列预留出空间*/
#container{
	padding-left: 200px;
	padding-right: 150px;
}

/*分别为三列设定宽度以及浮动*/
#container .column{
    float: left;
}

#center{
    width: 100%; /*必须设为100%，否则由于浮动具有包裹性，此时div的宽度会收缩*/
}

#left{
    width: 200px;
}

#right{
    width: 150px;
}

/*对底部的footer设置清除浮动，否则该元素会直接出现在header下方，因为浮动的元素已脱离文档流*/
#footer{
    clear:both;
}
```

此时结果如下：

{% asset_img 2.png %}

不难理解，因为`center`元素的宽度为100%（此处的百分比是指**子元素内容区域**相对于**父元素内容区域**，因此，保留了父元素中之前预留的两侧padding），所以`left`和`right`都在其下面。接下来再添加如下代码：

```css
#left{
	margin-left: -100%;
}
```

上述代码通过设置负外边距将`left`元素上移并覆盖住了`center`，可以看到如下图所示的结果。个人的理解是：当我们设置负外边距时，元素就会按照文档流反方向偏移（此处的三个浮动可以看作存在浮动流），当我们对于浮动元素设定`margin-left: -100%`，这里的100%相对的是上一个浮动元素的宽度，所以此时将左外边距设为-100%，就使得`left`元素产生了整体上移的效果。

{% asset_img 3.png %}

最后添加如下代码，使用相对定位并向左偏移使`left`元素来到正确的位置，对于`right`元素，同样使用负外边距，：

```css
#left{
	position: relative;
	right: 200px;
}

#right{
    margin-right: -150px;
}
```

{% asset_img 4.png %}

最后，为了保证布局效果正常显示，需要给页面添加一个最小宽度，但不是200 + 150 = 350，由于之前的`left`元素使用了`position:relative; right: 200px`，所以，最小宽度应为350 + 200 = 550。添加如下代码就大功告成了：

```css
body {
    min-width: 550px;
}
```

### 双飞翼布局

#### DOM结构

双飞翼布局的DOM结构如下，与圣杯布局有一点不同：`container`仅包含`center元素`，`column`也从`center`元素移动到了`container`上。

```html
<body>
    <div id="header">This is header</div>
    <div id="container" class="column">
        <div id="center">center</div>
    </div>
    <div id="left" class="column">left</div>
    <div id="right" class="column">right</div>
    <div id="footer">This is footer</div>
</body>
```

#### CSS布局

首先完成基本的颜色、宽度、浮动设定。此时不再通过`padding`预留空间（由于DOM结构的改变），而是通过`margin`实现：

```css
*{
    padding: 0;
    margin: 0;
}

#header, #footer{
    background-color: grey;
    padding: 5px;
    text-align: center;
}

#container{
    width: 100%;
}

#center{
    margin-left: 200px;
    margin-right: 150px;
    background-color: blueviolet;
    height: 250px;
}

.column{
    float: left;
}

#left{
    background-color: bisque;
    height: 290px;
    width: 200px;
}

#right{
    background-color: aqua;
    height: 300px;
    width: 150px;
}

#footer{
    clear: both;
}
```

得到结果如下，宽度为100%`container`占据了最上面一行，所以`left`和`right`元素都排在下面。**注意`center`并没有设置宽度和浮动**。

{% asset_img 5.png %}

增加以下代码：

```css
#left{
	margin-left: -100%;
}

#right{
	margin-left: -150px;
}
```

完成布局，得到和圣杯布局一样的效果：

{% asset_img 6.png %}

对于双飞翼布局，由于没有用到相对定位，页面宽度最小可以为350px， 但是当页面宽度缩小到350px附近时，会挤占中间栏的宽度，使得其内容被右侧栏覆盖， 所以在设置最小页面宽度时，可以适当增加一些宽度以供中间栏使用。

### 比较

+ 圣杯布局的DOM结构更加自然常见，只需要三个`div`即可。
+ **双飞翼布局通过在`center`外增加一个div的方式，改padding为margin，并使得不需要使用相对定位，更加简单**；
+ 由于没有了相对定位，页面的最小宽度也有没有了上文所述的限制。

我们注意到在圣杯布局中，对于`right`元素，最后使用了`margin-right: -150px;`，而在双飞翼中则是`margin-left: -150px;`来实现。个人的观点是，由于在圣杯布局中，我们的方式是通过`padding`内边距预留空间，如果我们此时使用`margin-left`就会导致如下结果：

{% asset_img 7.png %}

原因就是元素会紧挨着其在“元素流”（文档流或者浮动流）中的前一个元素，在圣杯布局中`right`的前一个浮动元素`center`，是在`container`的内容区里面，所以设置了负外边距后，不会按照预期的来到预留的位置，而是相对于`center`进入了内容区中。而在双飞翼中，它的前一个浮动元素是就是`container`，而`container`的宽度充满了整个屏幕，因此会得到正确结果。

**PS：其实在圣杯布局中，对于右栏使用`margin-left`也是可以的，然后再用一下相对定位的，和左栏一样（实际上大多数网上的圣杯实现左右栏都是使用了相对定位）**。至于为何使用`margin-right`会直接有这样的效果，目前我也还不是很清楚，<u>还望看到本博客的人不吝赐教</u>。

### 进阶：Flex布局

在CSS3引入Flex（弹性盒子）布局后，要实现这种分栏布局就变得更加简单了：

#### DOM结构

```html
<body>
    <div id="header">This is header</div>
    <div id="container">
        <div id="center">center</div>
        <div id="left">left</div>
    	<div id="right">right</div>
    </div>
    <div id="footer">This is footer</div>
</body>
```

#### CSS布局

```css
#container{
    display: flex;
}

#center{
    background-color: blueviolet;
    flex: 1;
}

#left{
    background-color: bisque;
    flex: 0 0 200px;
    order: -1;
}

#right{
    background-color: aqua;
    flex: 0 0 150px;
}
```

对上述代码做一个简单解释：要使用flex布局，首先将父容器的`display`属性设为`flex`，然后其下的所有子元素自动变成容器成员—我们称为项目。对于容器，我们可以设置属性规定容器中的项目的排列方式，而上述代码中，我们直接通过设定项目的属性：

+ `order`属性定义排列的属性，默认为0，我们将`left`的该属性设为`-1`，达到在最左边的目的。
+ `flex`属性是 `flex-grow`, `flex-shrink` 和 `flex-basis`的简写，默认值为`0 1 auto`。后两个属性可选。 
  + `flex-grow`属性定义项目的放大比例，默认为`0`，即如果存在剩余空间，也不放大。这里在`center`元素中将该值设为1，以实现自适应。
  + `flex-shrink`属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。这里在`left`和`right`元素中将该值设为0，保持宽度不变。
  +  `flex-basis`属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。默认值为`auto`，即项目的本来大小。主轴默认为横轴，这里即用来设定固定宽度大小。

此处对于Flex布局只做相关的简单介绍，因为自己也是不太明白其原理，希望然后可以了解的更加深入。

### 感想

CSS的世界真是千奇百怪，博大精深，作为刚开始学习的初学者，需要走的路还很多。

### 参考

[CSS布局奇淫巧计之-强大的负边距](https://www.cnblogs.com/2050/archive/2012/08/13/2636467.html)

[Flex 布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)