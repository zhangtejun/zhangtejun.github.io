---
layout: post
title:  "redis的发布订阅"
date:   2017-06-26 21:03:00
author: zhangtejun
categories: zhangtejun
---
##### Redis的发布订阅(用的较少,既不会用来做中间件) 主要还是用来分布式内存中间件缓存
进程间的一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

先订阅后发布后才能收到消息，
1. 可以一次性订阅多个，SUBSCRIBE c1 c2 c3
2. 消息发布，PUBLISH c2 hello-redis
3. 订阅多个，通配符*， PSUBSCRIBE new*
4. 收取消息， PUBLISH new1 redis2015
