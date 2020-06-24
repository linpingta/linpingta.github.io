---
layout: post
title: "Django Comment"
date: 2014-01-22 23:06:03 +0800
comments: true
categories: [Python,Django]
---

Django Comments
===

(下文因为octopress的生成语法，去掉了所有%)

Django自带Comments库，开启操作：
    
    1. 在INSTALLED_APPS中添加 django.contrib.comment 和 django.contrib.site （如果不加后者会出现'site'
    has a relation with model <class 'django.contrib.sites.model.Site'>）
    2. 更新数据库, manage.py syncdb
    3. 加入comments模块 (u'^comment/',include('django.contrib.comments.urls'))

在模板中使用comments

    load comments

查看当前评论 （comment都是基于object存在的）
    
    1. 查看评论数量： 
     get_comment_count for blog as comment_count 
    <p>Count {{coumment_count}}</p>
    
    2. 显示评论
     render_comment_list for blog 
    
    3. 自定义评论
     get_comment_list for blog as comment_list 
    
     for i in comment_list 
     endfor

    4. 显示评论提交表单
     get_comment_form for blog as form 
    
    <form action=" comment_form_target " method="post">
         csrf_token 
        {{ form }}
    </form>

comment_form_target 会返回comment系统中的提交对应的url (实际上url对应的views，views中的操作和存储都是在
comment系统中处理的)

"Setting" object has no attribute 'SITE_ID'

    在调用 get_comment_count 的时候出现以上错误，解决方案是在settings.py里面添加SITE_ID=1

Missing content_type or object_pk field

[解决方法](http://blog.csdn.net/bigconvience/article/details/7626437)

在csrf_token下面添加

         csrf_token 
        {{ form.content_type }}
        {{ form.object_pk }}
        {{ form.timestamp }}
        {{ form.security_hash }}

关于Django模板配置的原则：因为Django的模板允许继承，我通常用bare_base.html来构建结构，用base.html继承
bare_base.html来实现模块的通用功能，然后不同模块再继承base.html来实现各自不同的功能。

默认comments提交后，会跳转到comments默认的页面，此时处理方法有两种：
    
    1. 定制跳转页面
    在app目录下template/comment中重建跳转页面
    
    2. 强制跳转到另外一个页面
    默认的url是 /comment/posted/?c=1 所以在url中使其跳转到另外一个页面 （Django在做url
    匹配时，会跳转到第一个符合正则的页面）
    
    url(r'^comments/posted/','blog_redis.views.submit_comment',name='mySubmitComment'),
    
    然后再在submit_comment中实现跳转
    
    def submit_comment(request):
        return HttpResponseRedirect('/blog/archive')