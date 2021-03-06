---
layout: post
title:  "Zookeeper的安装与配置"
date:   2017-06-19 15:00:00
author: zhangtejun
categories: Zookeeper
---

##### Zookeeper

客户端注册监听它关心的目录节点，当这个结点发生变化（数据改变，被删除，子节点新增/删除），zk会通知给客户端。


zk四种形式的目录节点（默认是persistent ）
* 持久化目录节点（PERSISTENT）
* 持久化顺序编号目录节点（PERSISTENT_SEQUENTIAL）
* 临时目录节点（EPHEMERAL）
* 临时顺序编号目录节点（EPHEMERAL_SEQUENTIAL）


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
 #bin/zkServer.sh stop 停止
```

* Connecting to ZooKeeper
```shell
./zkCli.sh -server 127.0.0.1:2181
```

* From here, you can try a few simple commands to get a feel for this simple command line interface. First, start by issuing the list command, as in ls, yielding:

```shell
[zkshell: 8] ls /
[zookeeper]
```     
* Next, create a new znode by running create `/zk_test my_data`. This creates a new znode and associates the string "my_data" with the node. You should see:

```
[zkshell: 9] create /zk_test my_data
Created /zk_test
```   
* Issue another ls / command to see what the directory looks like:

```
[zkshell: 11] ls /
[zookeeper, zk_test]
```
        
Notice that the zk_test directory has now been created.

* Next, verify that the data was associated with the znode by running the get command, as in:

```
[zkshell: 12] get /zk_test
my_data
```       
We can change the data associated with zk_test by issuing the set command, as in:
```
[zkshell: 14] set /zk_test junk
`
[zkshell: 15] get /zk_test
junk
```
   
(Notice we did a get after setting the data and it did, indeed, change.

* Finally, let's delete the node by issuing:

```
[zkshell: 16] delete /zk_test
[zkshell: 17] ls /
[zookeeper]
[zkshell: 18]
```

