---
layout: post
title:  "配置文件redis.conf解析"
date:   2017-06-18 11:02:20
author: zhangtejun
categories: zhangtejun
---
##### redis.conf解析
1. units单位

    * 配置大小单位,开头定义了一些基本的度量单位，只支持bytes，不支持bit<br><br>   
	* 对大小写不敏感

2. INCLUDES包含

   和Struts2配置文件类似，可以通过includes包含，redis.conf可以作为总闸，可以包含其他conf文件。

3. GENERAL通用

	daemonize

	pidfile

	port

	tcp-backlog

	  设置tcp的backlog，backlog其实是一个连接队列，backlog队列总和=未完成三次握手队列 + 已经完成三次握手队列。在高并发环境下你需要一个高backlog值来避免慢客户端连接问题。注意Linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值，所以需要确认增大somaxconn和tcp_max_syn_backlog两个值来达到想要的效果

	timeout

	bind 

	tcp-keepalive

	单位为秒，如果设置为0，则不会进行Keepalive检测，建议设置成60

	loglevel

	logfile

	syslog-enabled

	是否把日志输出到syslog中

	syslog-ident

	指定syslog里的日志标志

	syslog-facility

	指定syslog设备，值可以是USER或LOCAL0-LOCAL7

	databases

4. SNAPSHOTTING快照

	Save

	 save 秒钟 写操作次数

	RDB是整个内存的压缩过的Snapshot，RDB的数据结构，可以配置复合的快照触发条件，<br><br>默认<br><br>是1分钟内改了1万次，<br><br>或5分钟内改了10次，<br><br>或15分钟内改了1次。

	禁用

	如果想禁用RDB持久化的策略，只要不设置任何save指令，或者给save传入一个空字符串参数也可以

	stop-writes-on-bgsave-error

	如果配置成no，表示你不在乎数据不一致或者有其他的手段发现和控制

	 rdbcompression

	rdbcompression：对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用<br><br>LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能

	 rdbchecksum

	rdbchecksum：在存储快照后，还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能

	 dbfilename

	 dir

5. REPLICATION复制

6. SECURITY安全

	访问密码的查看、设置和取消



7. LIMITS限制

	maxclients

	设置redis同时可以与多少个客户端进行连接。默认情况下为10000个客户端。当你无法设置进程文件句柄限制时，redis会设置为当前的文件句柄限制值减去32，因为redis会为自身内部处理逻辑留一些句柄出来。如果达到了此限制，redis则会拒绝新的连接请求，并且向这些连接请求方发出“max number of clients reached”以作回应。

	maxmemory

	设置redis可以使用的内存量。一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。如果redis无法根据移除规则来移除内存中的数据，或者设置了“不允许移除”，那么redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。但是对于无内存申请的指令，仍然会正常响应，比如GET等。如果你的redis是主redis（说明你的redis有从redis），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，才不用考虑这个因素

	maxmemory-policy

	 （1）volatile-lru：使用LRU算法移除key，只对设置了过期时间的键<br><br>（2）allkeys-lru：使用LRU算法移除key<br><br>（3）volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键<br><br>（4）allkeys-random：移除随机的key<br><br>（5）volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key<br><br>（6）noeviction：不进行移除。针对写操作，只是返回错误信息

	volatile-lru -> remove the key with an expire set using an LRU algorithm

	allkeys-lru -> remove any key according to the LRU algorithm

	volatile-random -> remove a random key with an expire set

	allkeys-random -> remove a random key, any key

	volatile-ttl -> remove the key with the nearest expire time (minor TTL)

	noeviction -> don't expire at all, just return an error on write operations

	maxmemory-samples

	设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小，redis默认会检查这么多个key并选择其中LRU的那个

8. APPEND ONLY MODE追加


	 appendonly

	 appendfilename

	appendfsync



	always：同步持久化 每次发生数据变更会被立即记录到磁盘  性能较差但数据完整性比较好

	everysec：出厂默认推荐，异步操作，每秒记录   如果一秒内宕机，有数据丢失


	no-appendfsync-on-rewrite：重写时是否可以运用Appendfsync，用默认no即可，保证数据安全性。

	auto-aof-rewrite-min-size：设置重写的基准值

	auto-aof-rewrite-percentage：设置重写的基准值

9. 常见配置redis.conf介绍

	参数说明<br><br>redis.conf 配置项说明如下：<br><br>1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程<br><br>  daemonize no<br><br>2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定pidfile /var/run/redis.pid<br><br>3. <br>指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字port 6379... 






