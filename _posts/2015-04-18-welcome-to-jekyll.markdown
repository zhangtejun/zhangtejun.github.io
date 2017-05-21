---
layout: post
title:  "Css 布局!"
date:   2017-05-21 10:25:39 +0800
categories: jekyll update
---
### "display"属性

`display` 是CSS中最重要的用于控制布局的属性。每个元素都有一个默认的 `display` 值，这与元素的类型有关。对于大多数元素它们的默认值通常是 `block` 或 `inline` 。一个 `block` 元素通常被叫做块级元素。一个 `inline` 元素通常被叫做行内元素。

#### block

`div` 是一个标准的块级元素。一个块级元素会新开始一行并且尽可能撑满容器。其他常用的块级元素包括 `p` 、 `form` 和HTML5中的新元素： `header` 、 `footer` 、 `section` 等等。

#### inline

`span` 是一个标准的行内元素。一个行内元素可以在段落中 <span> 像这样 </span> 包裹一些文字而不会打乱段落的布局。 `a` 元素是最常用的行内元素，它可以被用作链接。

#### none

另一个常用的`display`值是 `none` 。一些特殊元素的默认 `display` 值是它，例如 `script` 。 `display:none` 通常被 JavaScript 用来在不删除元素的情况下隐藏或显示元素。

它和`visibility` 属性不一样。把 `display` 设置成 `none` 元素不会占据它本来应该显示的空间，但是设置成 `visibility: hidden`; 还会占据空间。

每个元素都有一个默认的 `display` 类型。不过你可以随时随地的重写它！虽然“人为制造”一个行内元素可能看起来很难以理解，不过你可以把有特定语义的元素改成行内元素。常见的例子是：把 `li` 元素修改成 `inline`，制作成水平菜单。

### margin: auto;
{% highlight ruby %}
#main {
  width: 600px;
  margin: 0 auto; 
}
{% endhighlight %}
设置块级元素的 `width` 可以防止它从左到右撑满整个容器。然后你就可以设置左右外边距为 `auto` 来使其水平居中。元素会占据你所指定的宽度，然后剩余的宽度会一分为二成为左右外边距。

唯一的问题是，当浏览器窗口比元素的宽度还要窄时，浏览器会显示一个水平滚动条来容纳页面。让我们再来改进下这个方案...

### max-width
{% highlight ruby %}
#main {
  max-width: 600px;
  margin: 0 auto; 
}
{% endhighlight %}
在这种情况下使用 `max-width` 替代 `width` 可以使浏览器更好地处理小窗口的情况。这点在移动设备上显得尤为重要，调整下浏览器窗口大小检查下吧！

### 盒模型

在我们讨论宽度的时候，我们应该讲下与它相关的另外一个重点知识：盒模型。当你设置了元素的宽度，实际展现的元素却超出你的设置：这是因为元素的边框和内边距会撑开元素。看下面的例子，两个相同宽度的元素显示的实际宽度却不一样。
{% highlight ruby %}
.simple {
  width: 500px;
  margin: 20px auto;
}

.fancy {
  width: 500px;
  margin: 20px auto;
  padding: 50px;
  border-width: 10px;
}
{% endhighlight %}
### box-sizing
人们慢慢的意识到传统的盒子模型不直接，所以他们新增了一个叫做 `box-sizing` 的CSS属性。当你设置一个元素为 `box-sizing: border-box`; 时，此元素的内边距和边框不再会增加它的宽度。这里有一个与前一页相同的例子，唯一的区别是两个元素都设置了 `box-sizing: border-box`; ：
{% highlight ruby %}
.simple {
  width: 500px;
  margin: 20px auto;
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
}

.fancy {
  width: 500px;
  margin: 20px auto;
  padding: 50px;
  border: solid blue 10px;
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
}
{% endhighlight %}

既然没有比这更好的方法，一些CSS开发者想要页面上所有的元素都有如此表现。所以开发者们把以下CSS代码放在他们页面上：
{% highlight ruby %}
* {
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
}
{% endhighlight %}
这样可以确保所有的元素都会用这种更直观的方式排版。

不过 `box-sizing` 是个很新的属性，目前你还应该像我上面例子中那样使用 `-webkit-` 和 `-moz-` 前缀。这可以启用特定浏览器实验中的特性。同时记住它是支持IE8+的。

### position
为了制作更多复杂的布局，我们需要讨论下 position 属性。它有一大堆的值，名字还都特抽象，别提有多难记了。让我们先一个个的过一遍，不过你最好还是把这页放到书签里。

#### static
{% highlight ruby %}
.static {
  position: static;
}
{% endhighlight %}
`static` 是默认值。任意 `position: static`; 的元素不会被特殊的定位。一个 `static` 元素表示它不会被“positioned”，一个 `position` 属性被设置为其他值的元素表示它会被“positioned”。

#### relative
{% highlight ruby %}
.relative1 {
  position: relative;
}
.relative2 {
  position: relative;
  top: -20px;
  left: 20px;
  background-color: white;
  width: 500px;
}
{% endhighlight %}

`relative` 表现的和 `static` 一样，除非你添加了一些额外的属性。

在一个相对定位（`position`属性的值为`relative`）的元素上设置 `top 、 right 、 bottom` 和 `left` 属性会使其偏离其正常位置。其他的元素的位置则不会受该元素的影响发生位置改变来弥补它偏离后剩下的空隙。

#### fixed

一个固定定位（position属性的值为fixed）元素会相对于视窗来定位，这意味着即便页面滚动，它还是会停留在相同的位置。和 `relative` 一样， `top 、 right 、 bottom` 和 `left` 属性都可用。

{% highlight ruby %}
.fixed {
  position: fixed;
  bottom: 0;
  right: 0;
  width: 200px;
  background-color: white;
}
{% endhighlight %}
一个固定定位元素不会保留它原本在页面应有的空隙（脱离文档流）。

令人惊讶地是移动浏览器对 `fixed` 的支持很差。这里有相应的解决方案.

#### absolute

`absolute` 是最棘手的position值。 `absolute` 与 `fixed` 的表现类似，但是它不是相对于视窗而是相对于最近的“positioned”祖先元素。如果绝对定位（position属性的值为`absolute`）的元素没有“positioned”祖先元素，那么它是相对于文档的 body 元素，并且它会随着页面滚动而移动。记住一个“positioned”元素是指 position 值不是 static 的元素。

这里有一个简单的例子：
{% highlight ruby %}
.relative {
  position: relative;
  width: 600px;
  height: 400px;
}
.absolute {
  position: absolute;
  top: 120px;
  right: 0;
  width: 300px;
  height: 200px;
}
{% endhighlight %}
这个元素是相对定位的。如果它是 `position: static`; ，那么它的绝对定位子元素会跳过它直接相对于body元素定位。

absolute这个元素是绝对定位的。它相对于它的父元素定位。

通过具体的例子可以帮助我们更好地理解“position”。下面是一个真正的页面布局。
{% highlight ruby %}
.container {
  position: relative;
}
nav {
  position: absolute;
  left: 0px;
  width: 200px;
}
section {
  /* position is static by default */
  margin-left: 200px;
}
footer {
  position: fixed;
  bottom: 0;
  left: 0;
  height: 70px;
  background-color: white;
  width: 100%;
}
body {
  margin-bottom: 120px;
}
{% endhighlight %}
<img src="{{ site.logo | prepend: site.baseurl }}">



