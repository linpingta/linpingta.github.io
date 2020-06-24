---
layout: post
title: "Django Auth"
date: 2014-02-19 00:16:57 +0800
comments: true
categories: [Django,Python]
---

用户认证：采用Djanog内置的authorization. Django.contrib.auth
    
在urls.py里实现url和view的关联
    
    (r'^login/$','app.views.login_views',name='myLogin'),
    (r'^logout/$','app.views.logout_views',name='myLogout'),
    
在views.py中实现authorization操作
    
    def login_views(request):
        # user authenticate
        user = authenticate(user=request.POST['username'],password=request.POST['password'])
        if user is not None:
            login(request)
        else:
            print '#log fail#'
    
    def logout_views(request):
        logout(request)
        
    def index(request):
        if request.user.is_authenticated(): # if user logged in already
            # success operation
        else:
            # failed operation

在template中实现url跳转

    {% if user.is_authenticated %}
        <a class="btn danger small" href={ url 'myLogout' }>注销</a>
    {% else %}
        <div class="content-head-nav">
            <form action={ url 'myLogin' } method='post' class="content-head-form">
                { csrf_token }
                <input name='username' class="input-small" type="text" placeholder="用户名">
                <input name='password' class="input-small" type="password" placeholder="密码">
                <button class="btn btn-danger" type="submit">登录</button>
            </form>
        </div>
    {% endif %}  