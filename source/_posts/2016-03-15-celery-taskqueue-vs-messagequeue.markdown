---
layout: post
title: "Celery:任务队列 vs 消息队列"
date: 2016-03-15 19:58:23 +0800
comments: true
categories: [Python, Celery]
---


一. 什么是Celery
 
Celery是一个python模块,它在官网的[定义](http://www.celeryproject.org/)：

----------

    Celery is asynchronous task queue/job based on distributed message passing. It is focused on real-time operation, but supports scheduling as well.

这里强调的概念包括：异步任务队列 / 分布式消息传递 / 实时或调度任务,下面对这些概念分别做解释

二. 任务队列 vs 消息队列

消息队列作为进程间异步通信方式，在模块解耦上有广泛的应用。常用的消息队列包括: rabbitmq, zeromq, 以及redis等。
Celery首先在定义上强调了自己是task queue而不是message queue，同时它的具体实现上，会选择rabbitmq(默认方式)或者redis作为broker（貌似对其它mq和database也有支持，具体可见[broker列表](http://docs.celeryproject.org/en/latest/getting-started/brokers/index.html#broker-overview)，也说明它本身并不是一种消息队列，那么任务队列与消息队列的区别在哪里呢？

这个问题的回答主要参考[Quora/What is difference between a message queue and a task queue](https://www.quora.com/What-is-the-difference-between-a-message-queue-and-a-task-queue-Why-would-a-task-queue-require-a-message-broker-like-RabbitMQ-Redis-Celery-or-IronMQ-to-function)

截取较为重要的一段：

    Let's forget Celery for a moment. Let's talk about RabbitMQ. What would we usually do? Our Django/Flask app would send a message to a queue. We will have some workers running which will be waiting for new messages in certain queues. When a new message arrives, it starts working and processes the tasks

也就是说，任务队列会在一个比消息队列更高的层级管理你的工作，你不再需要关注消息的传递，而是重点定义工作的任务。

这仍然是一个模糊的概念，我们需要举一个具体的例子：这里我们定义两个task functions，表示Worker可以执行的任务，然后再在客户端调用执行这些任务。为了对比，我们再用rabbitmq作为消息队列，完成类似的功能。
(在这里省去Celery/rabbitmq/redis的安装和配置，我们假设相关环境已配置ok)。

Celery版本：
1. server端定义(tasks.py)

    from celery import Celery
    import time

    app = Celery('task', broker='redis://localhost',     backend='redis://localhost')
    app.conf.CELERY_TASK_SERIALIZER = 'json'

    @app.task
    def add(x, y):
        time.sleep(5)
        return x + y

    @app.task
    def multiply(x, y):
        return x * y

首先我们定义一个任务队列的实例app，它包含的参数：
'tasks':  任务名称，实践发现是可以不同于server文件名的，但简单处理可保持一致
broker:  指定Celery所依赖的消息队列地址，这里使用的是我本机的redis
backend: 指定Celery对tasks状态信息的存储位置，此处可以为空

简单对比下设置和未设置backend的效果（假设用redis作为backend):

1. 有backend设置运行task, server端日志：

>[2016-03-15 16:33:00,736: INFO/MainProcess] Received task: tasks.add[7e5c2132-d2a1-4ef6-a369-bb415b6499c4]
    
在redis中查找task_id：
>get celery-task-meta-7e5c2132-d2a1-4ef6-a369-bb415b6499c4
>"\x80\x02}q\x01(U\x06statusq\x02U\aSUCCESSq\x03U\ttracebackq\x04NU\x06resultq\x05K\bU\bchildrenq\x06]u."
2. 无backend设置运行task, server端日志：
>[2016-03-15 16:40:10,620: INFO/MainProcess] Received task: tasks.add[68a795c7-ab1e-4480-a735-3db2e8bd2918]

在redis中不能查找到这个task_id。

启动worker server:

    celery -A tasks worker --loglevel=info

可以看到类似如下启动日志：
>[tasks]
  . tasks.add
  . tasks.multiply
>[2016-03-15 16:40:03,028: INFO/MainProcess] Connected to redis://localhost:6379//
>[2016-03-15 16:40:03,122: INFO/MainProcess] mingle: searching for neighbors
>[2016-03-15 16:40:04,146: INFO/MainProcess] mingle: all alone
>[2016-03-15 16:40:04,188: WARNING/MainProcess] celery@iZ25d0yvrwwZ ready.

我们定义了两个任务add和multiply，redis作为broker对外提供服务。

2. 客户端定义(client.py)
异步调用

        from tasks import add
        import time

        result = add.delay(4, 4)
  
        while 1:
            if result.ready():
                print 'result ',result.get()
                break
            else:
                print 'not ready, wait 1 second'
                time.sleep(1)

因为我们在server端的add里面加入了sleep，因此result.ready()初始返回False，直到时间达到（模拟任务处理）返回处理结果，如下：

    not ready, wait 1 second
	not ready, wait 1 second
	not ready, wait 1 second
	not ready, wait 1 second
	not ready, wait 1 second
	result 8

同步调用
	
	# synchronous way
	try:
	    # short time
	    result.get(timeout=1)
	    # long time
	    result.get(timeout=10)
	except Exception as e:
	    print e
此时client会阻塞在get函数上，直到取到任务结果，或者timeout。

Rabbitmq版本

由于本篇主要介绍Celery功能，因此这里主要附上代码，它的基本原理是采用RPC调用实现消息传递

Server端 (add_consumer.py)

    import pika
	import simplejson as json
	
	connection = pika.BlockingConnection(pika.ConnectionParameters(
	        host='localhost'))
	
	channel = connection.channel()
	
	channel.queue_declare(queue='rpc_queue')
	
	def on_request(ch, method, props, body):
	    [x, y] = json.loads(body)
	
	    print(" [.] %d + %d" % (x, y))
	    response = x + y
	
	    ch.basic_publish(exchange='',
	                     routing_key=props.reply_to,
	                     properties=pika.BasicProperties(correlation_id = \
	                                                         props.correlation_id),
	                     body=str(response))
	    ch.basic_ack(delivery_tag = method.delivery_tag)
	
	channel.basic_qos(prefetch_count=1)
	channel.basic_consume(on_request, queue='rpc_queue')
	
	print(" [x] Awaiting RPC requests")
	channel.start_consuming()

Client端 (add_producer.py)

	import pika
	import uuid
	import simplejson as json
	
	class TaskRpcClient(object):
	    def __init__(self):
	        self.connection = pika.BlockingConnection(pika.ConnectionParameters(
	                host='localhost'))
	
	        self.channel = self.connection.channel()
	
	        result = self.channel.queue_declare(exclusive=True)
	        self.callback_queue = result.method.queue
	
	        self.channel.basic_consume(self.on_response, no_ack=True,
	                                   queue=self.callback_queue)
	
	    def on_response(self, ch, method, props, body):
	        if self.corr_id == props.correlation_id:
	            self.response = body
	
	    def add(self, x, y):
	        body = json.dumps([x, y])
	        self.response = None
	        self.corr_id = str(uuid.uuid4())
	        self.channel.basic_publish(exchange='',
	                                   routing_key='rpc_queue',
	                                   properties=pika.BasicProperties(
	                                         reply_to = self.callback_queue,
	                                         correlation_id = self.corr_id,
	                                         ),
	                                   body=body)
	        while self.response is None:
	            self.connection.process_data_events()
	        return int(self.response)
	
	task_rpc = TaskRpcClient()
	
	x = 4
	y = 4
	print(" [x] Requesting %d + %d" % (x,y))
	response = task_rpc.add(x, y)
	print(" [.] Got %r" % response)

这里的实现依据Rabbitmq[官方介绍](http://www.rabbitmq.com/tutorials/tutorial-six-python.html)。

通过上述代码的对比，可以看到，实现相同的功能，Celery要简洁很多。它对使用者隐藏了底层mq操作的复杂性，使得使用者可以以函数的形式更专注于实际任务，而不需要为mq操作过多操心。