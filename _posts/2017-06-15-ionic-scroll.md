---
layout: post
title:  "ionic滚动"
date:   2017-06-15 15:12:25
author: zhangtejun
categories: zhangtejun
---
# ionic scroll

##### ionic scroll 在ios中，当滚动至非webview时，无法响应touchend。
本文基于ionic1框架
1. 解决方案：封装自定义滚动指令。当滚动超出webview时，手动调用scrollTop(true)。

页面中：<ion-scroll my-scroll ....>...</ion-scroll>

```js
app.directive('myScroll', function($ionicScrollDelegate) {
	function link(scope, element, attrs) {
		var domScroll = (element[0]);
		var startX = 0, startY = 0;
		$this = domScroll;
		domScroll.addEventListener("touchstart", function(e) {
			startX = e.changedTouches[0].pageX;
			startY = e.changedTouches[0].pageY;
			console.log("start:(" + startX + "," + startY + ")");
		});
		domScroll.addEventListener("touchmove", function(e) {
			var x = e.changedTouches[0].pageX;
			var y = e.changedTouches[0].pageY;
			console.log("move:(" + x + startX + "," + y + startY + ")");
			//var aa = $ionicScrollDelegate.getScrollPosition();
			if (Math.abs(x - startX) > Math.abs(y - startY)) {//左右滑动 
				//scrollLeft($this,x);  
			} else if (Math.abs(y - startY) > Math.abs(x - startX)) { //上下滑动
				scrollTop($this, y);
			}
			function scrollLeft(obj, x) {
				e.preventDefault();
				e.stopPropagation();
			}
			function scrollTop(obj, y) {
				//var aa = $ionicScrollDelegate.getScrollPosition();

				var platformHeight = screen.height;// 设备屏幕高度

				var currentScrollHeight = platformHeight - 64 - 56;//设备webview高度

				//scope.test='startX-Y'+startX+','+startY+':'+'endX-Y'+x+','+y+'#'+currentScrollHeight;
				if (ionic.Platform.isIOS() && y > currentScrollHeight) {

					/**
					 *ion-scroll在ion-content里故需使用：scope.$parent来获取控制器里的属性
					 **/
					if ('function' === typeof (scope.$parent.doRefresh)) {//下拉刷新页面
						//手动触发下拉刷新
						scope.$parent.doRefresh();
					} else {
						//自动回弹至顶部/底部（scrollTop(true)）,true代表启用动画
						$ionicScrollDelegate.scrollTop(true);
					}
					e.preventDefault();
					e.stopPropagation();
				} else if (ionic.Platform.isIOS() && y < 0) {
					$ionicScrollDelegate.scrollBottom(true);
				}
			}
		});
	}
	return {
		restrict : 'AE',
		link : link
	};

});
```
##### jquery 摘自网络未验证
```js
.directive('myScroll',function(){
  function link($scope, element, attrs) { 
    var $domScroll=$(element[0]);
    var startX=0,startY=0;
    $domScroll.on("touchstart",function(e){
      startX=e.originalEvent.changedTouches[0].pageX;
      startY=e.originalEvent.changedTouches[0].pageY;
      console.log("start:("+startX+","+startY+")"+"--"+$(this).scrollTop());
    });
    $domScroll.on("touchmove",function(e){
      var x = e.originalEvent.changedTouches[0].pageX-startX;
      var y = e.originalEvent.changedTouches[0].pageY-startY;
      if ( Math.abs(x) > Math.abs(y)) {//左右滑动 
       scrollLeft($(this),x);  
      }else if( Math.abs(y) > Math.abs(x)){ //上下滑动
        scrollTop($(this),y);  
      }
      function scrollLeft(obj,x){ 
        var currentScroll = obj.scrollLeft();
        console.log(parseInt(currentScroll)-x);
        obj.scrollLeft(parseInt(currentScroll)-x);//滑动的距离
        e.preventDefault(); 
        e.stopPropagation();
      }
      function scrollTop(obj,y){ 
        var currentScroll = obj.scrollTop();
        console.log(parseInt(currentScroll)-y);
        obj.scrollTop(parseInt(currentScroll)-y);//滑动的距离
        e.preventDefault(); 
        e.stopPropagation();
      }
    });
  } 
  return {
    restrict: 'A',
    link: link 
  };
});

```

