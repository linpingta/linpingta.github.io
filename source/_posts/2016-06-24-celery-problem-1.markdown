---
layout: post
title: "Celery got an unexpected keyword argument 'socket_connect_timeout'"
date: 2016-06-24 10:52:17 +0800
comments: true
categories: [Python, Celery] 
keywords: 
description: 
---

这是一次问题的错误记录，celery在机器A上可以正常工作，但在机器B上启动时

	celery worker -A app --loglevel=info

会遇到如下问题：

	  File "/usr/local/lib/python2.7/site-packages/celery-3.1.23-py2.7.egg/celery/worker/consumer.py", line 376, in connect
	    callback=maybe_shutdown,
	  File "/usr/local/lib/python2.7/site-packages/kombu-3.0.35-py2.7.egg/kombu/connection.py", line 369, in ensure_connection
	    interval_start, interval_step, interval_max, callback)
	  File "/usr/local/lib/python2.7/site-packages/kombu-3.0.35-py2.7.egg/kombu/utils/__init__.py", line 246, in retry_over_time
	    return fun(*args, **kwargs)
	  File "/usr/local/lib/python2.7/site-packages/kombu-3.0.35-py2.7.egg/kombu/connection.py", line 237, in connect
	    return self.connection
	  File "/usr/local/lib/python2.7/site-packages/kombu-3.0.35-py2.7.egg/kombu/connection.py", line 742, in connection
	    self._connection = self._establish_connection()
	  File "/usr/local/lib/python2.7/site-packages/kombu-3.0.35-py2.7.egg/kombu/connection.py", line 697, in _establish_connection
	    conn = self.transport.establish_connection()
	  File "/usr/local/lib/python2.7/site-packages/kombu-3.0.35-py2.7.egg/kombu/transport/virtual/__init__.py", line 809, in establish_connection
	    self._avail_channels.append(self.create_channel(self))
	  File "/usr/local/lib/python2.7/site-packages/kombu-3.0.35-py2.7.egg/kombu/transport/virtual/__init__.py", line 791, in create_channel
	    channel = self.Channel(connection)
	  File "/usr/local/lib/python2.7/site-packages/kombu-3.0.35-py2.7.egg/kombu/transport/redis.py", line 464, in __init__
	    self.client.info()
	  File "/usr/local/lib/python2.7/site-packages/kombu-3.0.35-py2.7.egg/kombu/utils/__init__.py", line 325, in __get__
	    value = obj.__dict__[self.__name__] = self.__get(obj)
	  File "/usr/local/lib/python2.7/site-packages/kombu-3.0.35-py2.7.egg/kombu/transport/redis.py", line 908, in client
	    return self._create_client(async=True)
	  File "/usr/local/lib/python2.7/site-packages/kombu-3.0.35-py2.7.egg/kombu/transport/redis.py", line 861, in _create_client
	    return self.AsyncClient(connection_pool=self.async_pool)
	  File "/usr/local/lib/python2.7/site-packages/kombu-3.0.35-py2.7.egg/kombu/transport/redis.py", line 882, in __init__
	    self.connection = self.connection_pool.get_connection('_')
	  File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 455, in get_connection
	    connection = self.make_connection()
	  File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 464, in make_connection
	    return self.connection_class(**self.connection_kwargs)
	TypeError: __init__() got an unexpected keyword argument 'socket_connect_timeout'

网上关于此问题的主要讨论来自： 
https://github.com/celery/celery/issues/2903

但感觉大多数讨论并没有指明问题的解决方法。重新查看错误日志，看到问题最终出现在 /redis/connection.py中，即python redis客户端模块中，其中Connection类的构造函数如下：

	class Connection(object):
    "Manages TCP communication to and from a Redis server"
    description_format = "Connection<host=%(host)s,port=%(port)s,db=%(db)s>"

    def __init__(self, host='localhost', port=6379, db=0, password=None,
                 socket_timeout=None, encoding='utf-8',
                 encoding_errors='strict', decode_responses=False,
                 parser_class=DefaultParser):
        self.pid = os.getpid()
        self.host = host
        self.port = port
        self.db = db
        self.password = password
        self.socket_timeout = socket_timeout
        self.encoding = encoding
        self.encoding_errors = encoding_errors
        self.decode_responses = decode_responses
        self._sock = None
        self._parser = parser_class()
再对比可以运行机器中相应源代码：

	class Connection(object):
	    "Manages TCP communication to and from a Redis server"
	    description_format = "Connection<host=%(host)s,port=%(port)s,db=%(db)s>"
	
	    def __init__(self, host='localhost', port=6379, db=0, password=None,
	                 socket_timeout=None, socket_connect_timeout=None,
	                 socket_keepalive=False, socket_keepalive_options=None,
	                 retry_on_timeout=False, encoding='utf-8',
	                 encoding_errors='strict', decode_responses=False,
	                 parser_class=DefaultParser, socket_read_size=65536):
	        self.pid = os.getpid()
	        self.host = host
	        self.port = int(port)
	        self.db = db
	        self.password = password
	        self.socket_timeout = socket_timeout
	        self.socket_connect_timeout = socket_connect_timeout or socket_timeout
	        self.socket_keepalive = socket_keepalive
	        self.socket_keepalive_options = socket_keepalive_options or {}
	        self.retry_on_timeout = retry_on_timeout
	        self.encoding = encoding
	        self.encoding_errors = encoding_errors
	        self.decode_responses = decode_responses

可以看到在新版本的redis中才会支持socket_connect_timeout字段。

因为问题的解决办法是：升级redis到2.10以上版本

	pip install redis --upgrade
这个解决方案我也补充在 https://github.com/celery/celery/issues/2903 的最后了。s
