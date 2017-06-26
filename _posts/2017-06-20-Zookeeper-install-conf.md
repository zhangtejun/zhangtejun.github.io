---
layout: post
title:  "Zookeeper的安装与配置"
date:   2017-06-19 15:00:00
author: zhangtejun
categories: zhangtejun
---
##### Zookeeper的安装与配置

* 首先将安装包解压并进入该目录：
```shell
root@zhtjun:/home/zookeeper/zookeeper-3.5.3-beta#
```

* 修改系统的环境变量
```shell
root@zhtjun:/home/zookeeper/zookeeper-3.5.3-beta#vim /etc/profile
#ZooKeeper配置
export ZOOKEEPER_INSTALL=/home/zookeeper/zookeeper-3.5.3-beta
export PATH=$PATH:$ZOOKEEPER_INSTALL/bin
#export PATH=$JAVA_HOME/bin:$PATH jdk配置， 有何区别？
root@zhtjun:/home/zookeeper/zookeeper-3.5.3-beta# source /etc/profile
```
* 配置Zookeeper的配置文件
```shell
root@zhtjun:/home/zookeeper/zookeeper-3.5.3-beta#cd conf
root@zhtjun:/home/zookeeper/zookeeper-3.5.3-beta# cp zoo_example.cfg zoo.cfg
```
* 配置zoo.cfg
```shell
tickTime=2000 #ZooKeeper服务器心跳时间，单位为ms
initLimit=10 #投票选举心leader的初始化时间
syncLimit=5 #leader与follower心跳检测最大容忍时间，响应超过syncLimit*tickTime,
			#leader认为follower死掉，从服务器列表中删除follower
clientPort=2181 #端口
dataDir=/tmp/ZooKeeper/data #数据目录
dataLogDir=/tmp/ZooKeeper/log #日志目录
```

* 创建配置中的相应目录
```shell
#mkdir -p /tmp/ZooKeeper/data
cd /tmp
mkdir ZooKeeper
cd Zookeeper
mkdir log
mkdir data
```
* 启动ZooKeeper
```shell
cd /usr/ZooKeeper/bin
./zkServer.sh start

# bin/zkServer.sh stop 停止
```
* Connecting to ZooKeeper
```shell
./zkCli.sh -server 127.0.0.1:2181
```