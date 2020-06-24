---
layout: post
title: "Celery: 建议资料汇总"
date: 2016-03-20 10:02:52 +0800
comments: true
categories: [Python, Celery] 
keywords: 
description: 
---

这篇博客是网上关于Celery的一些使用建议方面的资料汇总。

[CELERY - BEST PRACTICES](https://denibertovic.com/posts/celery-best-practices/)
1. Don't use the database as your AMQP Broker
不要用数据库作为AMQP的broker，虽然Celery支持这样做，但是数据库可能会因此导致更差的IO性能，进而影响实际提供的服务
2. Use more Queues (ie. not just the default one)
把不同的任务扔到不同的队列中处理，因为他们的处理时间和优先级不同
3. Use priority workers
同2.
4. Use Celery's error handling mechanisms
在任务函数的定义中，注意异常处理，包括try...except,  设置重试次数和重试间隔
5. Use Flower
Flower: [Real-time Celery web-monitor](http://celery.readthedocs.org/en/latest/userguide/monitoring.html#flower-real-time-celery-web-monitor)
6. Keep track of results only if you really need them
只有在你需要的时候才关注任务执行的结果，设置CELERY_IGNORE_RESULT = True
7. Don't pass Database/ORM objects to tasks
不要把ORM对象传入task

[Three quick tips from two years with Celery](https://library.launchkit.io/three-quick-tips-from-two-years-with-celery-c05ff9d7f9eb#.ewpt0c9eo)
1. Set a global task timeout
默认情况下，task不会结束，有必要设置全局的timeout时间，以免由于网络或者其它原因出现无法结束的任务。
2. Use -Ofair for your preforking workers
任务会在worker启动后分配，而不是预先分配，避免某个task执行时间过长导致worker的利用不充分
3. Use exponential retry delays
类似上一篇文章的第4项，且看起来没有它优雅

[7 Tips for Working with Celery](https://blog.jixee.me/7-tips-for-working-with-celery/)
1. Set up Task Timeout
设置任务的超时时间，同上
2. Set the Number of Clients
意义不大
3. -Ofair Task Distribution for Preforking Workers
同上
4. Setup Retry Delays
同上
5. JSON Serializer for Inoperability
用JSON作为序列化方式
6. Celery In Development
7. Configure a Broker
意义不大