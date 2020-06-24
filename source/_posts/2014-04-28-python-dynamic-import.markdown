---
layout: post
title: "Python 模块动态加载"
date: 2014-04-28 22:20:16 +0800
comments: true
categories: [Python]
---

这篇博客的缘由是在stackoverflow上回答的一个相关[问题](http://stackoverflow.com/questions/23210646/a-dynamic-load-import-reports-no-module-error/23232976#23232976)

#为什么要引入动态加载
假如我们有如下的一个main函数

in main.py
    
    import Command1
    import Command2
    ...
    import CommandN
    
    if __name__ == '__main__':
        if condition1:
            Command1.run()
        elif:
            Command2.run()
        ...
        else:
            CommandN.run()

我们需要根据一定的条件调用某模块下的函数，但是我们只需要其中的一部分。在这种情况下，把其它模块均加载进去并非有必要。动态加载可以解决这个问题。

#引入动态加载的方式 \__import__

\__import__是python内建函数，它允许用户输入字符串，然后查找对应名称的模块并返回。
例如：
    
    sys = __import__('sys')
和

    import sys
得到的sys是一致的。

然而，当模块中包含多个'.'的时候，（如'foo.foo1.Command1'），那么简单使用 __import__ 不能正确解析模块，原因在于 __import__ 会将第一个逗号后面的内容视为模块名称。

关于这一问题有下面两种解决方法：

1. 使用fromlist参数：
    

     __import__("foo.foo1.Command1", fromlist=["foo.foo1"])

2. 使用importlib模块：
    

    import importlib
    importlib.import_module("foo.foo1.Command1")