---
layout: post
title:  "js时间"
date:   2017-06-17 09:12:45
author: zhangtejun
categories: zhangtejun
---
##### js表单校验
1. 表单元素直接绑定事件处理函数
```html
//check 返回false将会阻止表单提交
<form onsubmit="return check(this)"><form>
```
2. 绑定DOM对象属性来设置事件处理函数
```js
//check 返回false将会阻止表单提交
document.forms[0].onsubmit = check;
```
[HTML 事件属性](http://www.w3school.com.cn/tags/html_ref_eventattributes.asp)

##### 使用返回值改变属性默认行为
1. 事件处理函数返回值方式
```html
//阻止超链接导航
<a href ="http://www.baidu.com" onclick="return false" />
```
2. 事件对象的returnValue属性设置为false
```
<script>
function clickHandler(){
//doSomething
event.returnValue = fasle;
}
<script/>
```

##### spript for IE浏览器独有的绑定事件机制
```js
<input  type="button"  value="button" id="bt1"/>
<!--  使用spript for 将以下脚本绑定到bt1按钮的onclick事件 -->
<script for="bt1" event="onclick">
//doSomething
<script/>
```
##### IE
```
/**
* eventName:事件名，functionReference:函数引用
**/
domObj.attachEvent("eventName", functionReference); 
//删除事件绑定
domObj.detachEvent("eventName", functionReference); 
```
一个DOM对象，一次事件可以多次绑定。先绑定后执行。

##### 事件冒泡
IE中事件的传递方向是从事件发生的对象开始，即从下先上传递，因而称为冒泡。
```
//event事件对象. 执行顺序showMsg2()--->showMsg()
<div onclick="showMsg(this,event)" id="outSide">
	<div onclick="showMsg2(this,event)" id="inSide" ></div>
</div>
<script type="text/javascript">
function showMsg(obj,e)
{
    alert(obj.id);
}
</script>
```

阻止事件冒泡，可以修改event的cancelBubble属性，默认为false.
```
//仅执行showMsg2()
<div onclick="showMsg(this,event)" id="outSide">
	<div onclick="showMsg2(this,event);event.cancelBubble=true" id="inSide" ></div>
</div>
```
##### 重定向事件
事件冒泡机制严格从子节点向父结点上溯。如果需要在不同节点将跳跃，可以使用重定向事件来实现。
IE  将event事件重定向到target对象
```
target.fireEvent(String eventType, Event event);
```

#### DOM的事件模型
##### DOM事件绑定
addEventListener和removeEventListener两个函数。
```
//objTarget绑定事件handler，eventType为(click等)，handler为事件处理函数，
//captureFlag 指定检讨事件传播的哪个阶段(true:监听捕获阶段-->由外向内触发，false:监听冒泡阶段-->由内向外触发)。
objTarget.addEventListener("eventType",handler,captureFlag);
```
##### 访问事件对象
DOM事件模型和IE事件模型访问事件对象方式不同，在DOM事件模型中，当浏览器检测到发送某个事件时，将自动创建一个Event对象，
并隐式将该对象作为事件处理函数的第一个参数传入。

跨浏览器
```js
<button id="bt1"  >bt1</button>
<script type="text/javascript">
//定义行参evt  该行参最好不要为event,可能导致该行参覆盖全局可用的event对象，导致IE不能访问该属性。
var clickHandler = function(evt){
	//对应DOM,访问事件源用target属性
	if(evt){
		alert(evt.target.innerHTML);
	}else{ //IE
		alert(window.event.srcElement.innerHTML);
	}
}
document.getElementById("bt1").onclick = clickHandler;
</script>
```

##### 事件传播
调用事件对象的stopPropagation()方法以阻止事件的继续传播。
event.stopPropagation():阻止event事件传播。

event.preventDefault():取消event事件的默认行为。
```
<a href="http://url" id="link">导航</a>
<script>
var clickHandler = function(event){
	//取消event事件的默认行为，但是不阻止事件传播
	event.preventDefault();
	alert("link");//结果：弹出link,但是不导航至http://url。
}
document.getElementById("link").addEventListener("click",clickHandler ,true);//捕获阶段
</script>
```