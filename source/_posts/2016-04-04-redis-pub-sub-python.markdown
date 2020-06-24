---
layout: post
title: "redis-pub-sub-python"
date: 2016-04-04 16:37:41 +0800
comments: true
categories: [Python,Redis] 
keywords: redis pub/sub python
description: 
---


Redis Pub/Sub

一.定义
[pubsub](http://redis.io/topics/pubsub)指的是发布者将消息发布到指定通道，并不清楚（或者说不关心）订阅者，同时订阅者也指定接收通道接收消息，并不关心具体的发布者。通过这样的方法，实现发布者和订阅者之间的解耦。

二.Redis原语

1.订阅channe1到channel3的三个频道：		

    subscribe channel1 channel2 channel3

2.向channel1发布消息"hello_world"：

	publish channel1 "hello_world"

3.channel1接收消息格式：

	message channel1 REAL_MESSAGE
其中第一项是消息类别，第二项是频道名称，第三项是实际接收的消息

4.unsubscribe结束对所有频道的订阅，在客户端调用

5.pattern-matching pub/sub
根据正则做消息订阅，接收所有以news开头频道的消息：

psubscribe news*
如果这里用的是subscribe，那么只能接收到名为"news*"频道的消息，而不是以news开头所有频道的消息。

三.python版本实现

1.pubilsh.py
实现发布的功能是比较简单的（当然也是在原型条件下，没有考虑接收者的消息反馈和重发），代码基本只有三行：

    import redis

	r = redis.Redis(host='localhost')
	r.publish('ch1', 'hello world')

这里publish的返回值是当前这个频道subscribe的数量

2.阻塞subscribe.py
实现阻塞版本的消息接收需要使用listen函数：

    import redis

	r = redis.Redis(host='localhost')
	pubsub = r.pubsub()
	pubsub.subscribe('ch1')
	
	for item in pubsub.listen():
	    print item
如果需要实现轮询的效果，可以在for外面再套一层while循环；listen的返回值格式如下：

    {'pattern': None, 'type': 'message', 'channel': 'ch1', 'data': 'hello,wolrd'}

其中有效部分为item["data"]。

3.非阻塞subscribe.py
在subscribe的接口层面，这是不成立的，但是可以在语言层面通过多线程实现：独立的线程负责接收接口消息，主线程可以做自己的事情。

    import redis
	import threading
	import time
	
	def subscriber():
	    r = redis.Redis(host='localhost')
	    sub = r.pubsub()
	    sub.subscribe('a1')
	    while True:
	        for item in sub.listen():
	            print item
	
	
	if __name__ == '__main__':
	
	    t = threading.Thread(target=subscriber)
	    t.setDaemon(True)
	    t.start()
	    while 1:
	        print 'waiting'
	        time.sleep(10)
