---
layout: post
title: "Celery常见问题"
date: 2016-06-28 09:43:58 +0800
comments: true
categories: [Python, Celery] 
keywords: 
description: 
---

问题节选自[Frequently Asked Questions](http://docs.celeryproject.org/en/latest/faq.html)，只挑选了一些感觉比较重要的回答，也掺杂了一些自己的理解，有兴趣请阅读[原文](http://docs.celeryproject.org/en/latest/faq.html)。

1.Celery的适用场景

	(1) 后台运行的场景。比如web应用中，需要快速返回结果给用户，以求得更好的用户体验，但任务本身还需要在后台运行，此时很适合使用Celery。
	(2) 周期性执行的任务
还有几点可以视为Celery的属性

	(1) 异步执行，包含重试
	(2) 分布式
	(3) 并行

2.Celery的依赖

作者在这里吐槽了下对依赖过多的批评："The rationale behind such a fear is hard to image"，软件本身应该避免重复造轮子，而且有像pip这样的包管理工具，安装依赖本身并不是一件头疼的事情。

这里我想吐槽下依赖的问题：最近在工作里用到Celery发现启动后遇到[Socket_connect_timeout错误](http://linpingta.cn/blog/2016/06/24/celery-problem-1/)，最终排查后，发现是kombu的版本略新，而redis的版本略旧导致了问题，所以说依赖如果能够保证版本一致还好，但如果同一模块不同版本间升级较多，依赖者就要麻烦了。

回过头来看看Celery的依赖：

	(1) kombu：一个python消息库，据说在openstack项目中也有使用
	(2) billard：一个对python自带multiprocessing模块功能升级的模块
	(3) pytz：时区管理
另外Django也提供Celery的使用支持。

3.Celery算是一个轻量级服务吗
是的，但同样需要考虑CPU和IO的[优化]。(http://docs.celeryproject.org/en/latest/userguide/optimizing.html#guide-optimizing)

4.Celery的序列化方式
默认是pickle，但同样支持yaml, json等其它格式。

5.Celery适用的web框架
不仅是Django，也包括其它

6.Celery的broker选择
推荐使用RabbitMQ或者支持AMQP协议的消息队列，但是Redis，或者其它数据库（MongoDB, 关系型数据库）也是可以选择的。
补充下，在这里Redis的支持应该是好于其它数据库（无论是关系还是非关系），另外也有文章说最好不要用关系型数据库作为Celery的broker，所以其实主要的选择就是两点：RabbitMQ或者Redis。
像我们工作的环境，RabbitMQ并没有安装，因此我们选择Redis作为broker。

7.Celery支持多语言么
支持
这点我还没有什么应用，这里不乱说了，有兴趣请看[原文](http://docs.celeryproject.org/en/latest/faq.html)。

8.如何清除队列里的任务
调用purge命令。
如果是清除所有任务，可以：

	celery -A proj purge
如果是清除指定队列中的任务，可以：

	celery -A proj amqp queue.purge <queue name>

9.如何根据taskID获取任务执行的结果
调用task.AsyncResult命令

	result = my_task.AsyncResult(task_id)
	result.get()

10.可以以root身份运行celery么
一定不要这样，可能的安全问题会导致风险。

11.如何在一个任务调用完后执行另外一个任务
依赖[Canvas](http://docs.celeryproject.org/en/latest/userguide/canvas.html)中定义的操作符，在执行完A后执行B：

	do A | do B

12.如何取消正在执行的任务

	result = add.apply_async(args=[2, 2], countdown=120)
	result.revoke()

13.如何将指定任务发送到指定server
	使用[路由](http://docs.celeryproject.org/en/latest/userguide/routing.html)

14.Celery支持优先操作么
不支持消息级别的优先级操作，Celery对优先级操作的支持是通过把不同优先级的任务发送到不同的任务路由实现。

15.可以指定时间执行任务么
类似crontab的调用：

	from celery.schedules import crontab
	from celery.task import periodic_task
	
	@periodic_task(run_every=crontab(hour=7, minute=30, day_of_week="mon"))
	def every_monday_morning():
	    print("This is run every Monday morning at 7:30")