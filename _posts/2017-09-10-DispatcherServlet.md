---
layout: post
title:  "DispatcherServlet"
date:   2017-09-10 10:29:00
author: zhangtejun
categories: zhangtejun
---
##### DispatcherServlet
用于HTTP请求处理程序/控制器的中央调度程序。

Web应用程序可以定义任意数量的DispatcherServlet。 每个servlet将在自己的命名空间中运行，
使用映射，处理程序等加载自己的应用程序上下文。只有通过加载的根应用程序上下文 ContextLoaderListener才能被共享。
