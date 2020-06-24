---
layout: post
title: "Django Setting"
date: 2014-01-02 23:04:15 +0800
comments: true
categories: [Python,Django]
---

Django配置
===

静态文件处理 

http://blog.sina.com.cn/s/blog_3fe961ae01016fpk.html

Django会先在STATICFILES_DIRS配置的目录下查找静态文件，如果不存在，在每个app的static子目录（如果存在）内查找

    PROJECT_ROOT = os.path.abspath(os.path.join(os.path.dirname(__file__),'..'))
    
    STATICFILES_DIRS = (
        os.path.join(PROJECT_ROOT, 'static'),
    )

另外STATIC_ROOT是在部署环境下指定静态文件的存储位置

    STATIC_ROOT = (
        os.path.jon(PROJECT_ROOT, 'asset')
    )