---
layout: post
title: "Publish Django Website"
date: 2014-03-24 18:29:43 +0800
comments: true
categories: [Django,Apache]
---

Django生产环境配置

一. Django + Apache

Django本身提供了一套开发测试服务器，但考虑到实际应用的需求（并发性的支持），需要将Django部署到Apache上。下面是部署的操作步骤 

假设你已经安装了[Django](https://www.djangoproject.com/download/)和[Apache](http://httpd.apache.org/download.cgi),我使用的是Django1.6.2和Apache2.2

1.安装mod_wsgi

根据官网介绍，"The aim of mod_wsgi s to implement a simple to use Apache module which can host any Python application which supports the Python WSGI interface"。mod_wsgi主要支持Python应用在Apache上面的部署。
在官网上支持的源代码下载[mod_wsgi](http://modwsgi.googlecode.com/files/mod_wsgi-3.3.tar.gz)，需要自己编译产生mod_wsgi.so，如果在windows上出于方便考虑也可以直接使用编译好的[二进制文件](http://www.lfd.uci.edu/~gohlke/pythonlibs/#mod_wsgi)
将生成的mod_wsgi.so文件拷贝到Apache目录的module文件夹内。同时在http.conf中加载该模块

    LoadModule wsgi_module modules/mod_wsgi-win32-ap22py27-3.3.so

2.配置Django应用
在Django app的根目录创建一个文件夹apache，其中设置配置文件django.wsgi (也可以不建文件夹直接把django.wsgi放在外面)，django.wsgi文件内容如下：

    import os
    import sys
    
    path = 'F:/GitHub/xxx' # replace this to your dir
    if path not in sys.path:
        sys.path.append(path)
    	
    os.environ['DJANGO_SETTINGS_MODULE'] = 'xxx.settings' # path of settings.py
    
    import django.core.handlers.wsgi
    application = django.core.handlers.wsgi.WSGIHandler()

其中需要修改的内容是path和django setting路径，这里请修改成你自己的

3.配置http.conf
VirtualHost是在同一台机器上搭建属于不同域名或者基于不同IP的网站，设置的标准格式如下（即http.conf中的注释内容）

    # <VirtualHost *:80>
    #    ServerAdmin webmaster@dummy-host.example.com
    #    DocumentRoot /www/docs/dummy-host.example.com
    #    ServerName dummy-host.example.com
    #    ErrorLog logs/dummy-host.example.com-error_log
    #    CustomLog logs/dummy-host.example.com-access_log common
    #</VirtualHost>
其中ServerAdmin指定网站的域名，DocumentRoot指定网站的根目录，如果需要多个应用，那么简历多个VirtualHost。因为我并不打算在计算机上配置多个应用，因此我并没有设置VirtualHost。
我在http.conf中的设置主要有三部分：
1. 设置WSGIScriptAlias，WSGIScriptAlias指定包含WSGI脚本的路径，使得mod_wsgi在加载处理时能够调用python的相关应用。
    
    WSGIScriptAlias / F:/GitHub/xxx/apache/django.wsgi
2. 设置Django程序的访问路径权限

    <Directory F:/GitHub/muni.python/apache/>
    	Options +ExecCGI
    	Order deny,allow
    	Allow from all 
    </Directory>
3. 设置静态文件路径及权限

    Alias /static/ F:/GitHub/xxx/static/
    <Directory F:/GitHub/xxx/static/>
    	Order deny,allow
    	Allow from all 
    </Directory>

完成以上的设置，重启Apache，通过http://localhost:80访问对应Django urls.py中 url(r'^$','app.views.index',name='index')对应的url

二. 外网访问

如果只是部署好服务器，那么只能支持内网的访问，外网仍然无法进行访问。我用笔记本连家里的无线路由宽带上网，那么需要如下的配置:
1. 设置无线路由的映射
重启无线路由，在配置中“转发规则”->"虚拟服务器"->"添加新条目"，把添加无线路由的IP （比如192.168.1.100）
查询自己的[真实IP](http://www.ip138.com/)，比如123.123.123.123

2. 设置非80端口
这点操作比较简单，主要是好像对联通80等标准端口会被屏蔽，因此在http.conf中设置一个非常用的端口，比如我设置的是30509

外网访问 http://123.123.123.123:30509/ 完成
