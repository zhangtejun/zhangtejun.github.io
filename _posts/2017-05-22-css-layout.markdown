---
layout: post
title:  "Css 布局!"
date:   2017-05-22 10:25:20
author: zhang
categories: zhang
---
#### CSS 基础知识参考
<ul>
<li><a href="http://www.imooc.com/learn/9">幕课网 - HTML+CSS基础课程</a>：偏基础，可以在线练习和预览</li>
<li><a href="https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Getting_started">MDN - CSS入门教程</a>: MDN 的官方文档</li>
<li><a href="http://zh.learnlayout.com/">学习 CSS 布局</a>：排版和配色特别舒服，简短但不深入，适合概览入门</li>
</ul>
#### CSS 定位问题
主要就是经典的绝对定位，相对定位问题
<ul>
<li><a href="http://www.barelyfitz.com/screencast/html-training/css/positioning/">10个文档学布局</a>：通过十个例子讲解布局，主要涉及相对布局，绝对布局，浮动。</li>
<li><a href="http://ife.baidu.com/note/detail/id/662">百度前端学院笔记 - 理解绝对定位</a>：文章本身一般，几篇参考文献比较详细</li>
<li><a href="http://www.w3cplus.com/css/advanced-html-css-lesson2-detailed-css-positioning.html">HTML和CSS高级指南之二——定位详解</a>（译文）：介绍浮动的使用，详细介绍定位的技巧，包括如何准确的给元素在 X 轴、Y 轴和 Z 轴定位</li>
</ul>

### 三栏式布局

涉及浮动和清除浮动，主要讲解“圣杯”和“双飞翼”两种解决方法。这两种方法实现的都是三栏布局，两边的盒子宽度固定，中间盒子自适应，它们实现的效果是一样的，差别在于其实现的思想。

#### 圣杯布局

圣杯：父盒子包含`三个子盒子`（左，中，右）

* 中间盒子的宽度设置为 `width: 100%;` 独占一行；
* 使用负边距(均是 `margin-left`)把左右两边的盒子都拉上去和中间盒子同一行；
	* `.left {margin-left:-100%;}` 把左边的盒子拉上去
	* `.right {margin-left：-右边盒子宽度px;}` 把右边的盒子拉上去,需要设置其左边距为负的自己的宽度(`盒子也需要考虑边框的宽度`)
* 父盒子设置左右的 `padding` 来为左右盒子留位置；`{ padding: 0  200px;}`200px是左右盒子宽度
* 对左右盒子使用相对布局来占据 `padding` 的空白，避免中间盒子的内容被左右盒子覆盖；
* 给左右两个盒子加一个相对定位，加了定位之后左右两个盒子就可以设置left和right值。`.left{ position: relative; left: -200px;}`

{% highlight ruby %}
<!-- 圣杯的 HTML 结构 -->
<div class="container">
    <!-- 中间的 div 必须写在最前面 -->
    <div class="middle">中间弹性区</div>
    <div class="left">左边栏</div>
    <div class="right">右边栏</div>
</div>
{% endhighlight %}

#### 双飞翼布局

双飞翼：父盒子包含`三个子盒子`（左，中，右），中间的子盒子里再加一个`子盒子`。

* 中间盒子的宽度设置为 `width: 100%;` 独占一行；
* 使用负边距(均是 `margin-left`)把左右两边的盒子都拉上去和中间盒子同一行；(同上圣杯布局)
* 在中间盒子里面再添加一个 `div`，然后对这个 `div` 设置 `margin-left` 和 `margin-right`来为左右盒子留位置；
{% highlight ruby %}
<!-- 双飞翼的 HTML 结构 -->
<div class="container">
    <!-- 中间的 div 必须写在最前面 -->
    <div class="middle">
         <div class="middle-inner">中间弹性区</div>
    </div>
    <div class="left">左边栏</div>
    <div class="right">右边栏</div>
</div>
{% endhighlight %}

#### 圣杯和双飞翼异同

圣杯布局和双飞翼布局解决的问题是一样的，都是两边定宽，中间自适应的三栏布局，中间栏要在放在文档流前面以优先渲染。

两种方法基本思路都相同：首先让中间盒子 100% 宽度占满同一高度的空间，在左右两个盒子被挤出中间盒子所在区域时，使用 margin-left 的负值将左右两个盒子拉回与中间盒子同一高度的空间。接下来进行一些调整避免中间盒子的内容被左右盒子遮挡。

主要区别在于 如何使中间盒子的内容不被左右盒子遮挡：
* 圣杯布局的方法：设置父盒子的 padding 值为左右盒子留出空位，再利用相对布局对左右盒子调整位置占据 padding 出来的空位；
* 双飞翼布局的方法：在中间盒子里再增加一个子盒子，直接设置这个子盒子的 margin 值来让出空位，而不用再调整左右盒子。
* 简单说起来就是双飞翼布局比圣杯布局多创建了一个 div，但不用相对布局了，少设置几个属性。

### 居中布局
#### 水平居中
* 对于行内元素(inline)：`text-align: center;`
* 对于块级元素(block)：设置宽度且 `marigin-left` 和 `margin-right` 是设成 `auto`
* 对于多个块级元素：对父元素设置 `text-align: center;`，对子元素设置 `display: inline-block;`；或者使用 `flex` 布局
#### 垂直居中
* 对于行内元素(inline)
	* 单行：设置上下` pandding` 相等；或者设置 `line-height` 和 `height` 相等
	* 多行：设置上下 `pandding` 相等；或者设置 `display: table-cell;` 和 `vertical-align: middle;`；或者使用 `flex `布局；或者使用伪元素
	* 对于块级元素(block)：下面前两种方案，父元素需使用相对布局
	* 已知高度：子元素使用绝对布局 `top: 50%;`，再用负的 `margin-top` 把子元素往上拉一半的高度
	* 未知高度：子元素使用绝对布局 `position: absolute; top: 50%; transform: translateY(-50%);`
	* 使用 `Flexbo`：选择方向，`justify-content: center;`
水平垂直居中
	* 定高定宽：先用绝对布局 `top: 50%; left: 50%;`，再用和宽高的一半相等的负 `margin` 把子元素回拉
	* 高度和宽度未知：先用绝对布局 `top: 50%; left: 50%;`，再设置 `transform: translate(-50%, -50%);`
	* 使用 `Flexbox：justify-content: center; align-items: center;`

### Flexbox布局









