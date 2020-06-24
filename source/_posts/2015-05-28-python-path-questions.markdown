---
layout: post
title: "python 路径相关的小问题"
date: 2015-05-28 16:39:48 +0800
comments: true
categories: [Python] 
keywords: python; path;
description: 
---

只要是稍微大一些的python项目，肯定就不会只包含一个python文件。这篇博客就是关于项目中python文件管理的一些概念说明。

1. \__init\__.py文件
在directory中包含此文件，表示这个directory是一个module。假设我们的directory名称为lib, 里面包含子文件file1.py, file2.py，那么如果想在其它文件中访问file1.py中的类或者函数，需要两个步骤：

	1. 添加lib目录为module，即在其中创建一个空的__init__.py文件
	2. 添加lib目录到sys.path，使得调用对象可以在搜索目录中找到此模块

一旦目录可访问，调用者可以通过

	from file1 import * 
来访问其中定义的函数和类

2.sys.path
直接在py文件中打印sys.path，可能会得到类似下面的目录列表：

	['/home/test/path_test', '/usr/local/lib/python2.7/dist-packages/mccabe-0.3.1-py2.7.egg', '/usr/local/lib/python2.7/dist-packages/pep8-1.5.7-py2.7.egg', '/usr/local/lib/python2.7/dist-packages/pyflakes-1.0.0-py2.7.egg', '/usr/lib/python2.7', '/usr/lib/python2.7/plat-x86_64-linux-gnu', '/usr/lib/python2.7/lib-tk', '/usr/lib/python2.7/lib-old', '/usr/lib/python2.7/lib-dynload', '/usr/local/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages/PILcompat', '/usr/lib/python2.7/dist-packages/gtk-2.0', '/usr/lib/pymodules/python2.7']

其中第一项（即sys.path[0])是调用文件的父目录，后面的所有项来自PYTHONPATH环境变量的定义。
仍然举上面lib的例子，如果调用文件test.py和lib目录平级，那么因为sys.path[0]包含了当前目录，通过

	from lib.file1 import *
可以直接访问file1中定义的类和函数。
但是如果调用文件test.py定义在一个与lib目录平级的目录（比如bin目录）中，那么直接按上面书写，因为lib目录并不是bin目录的子目录，因此python解释器不知道如何找到该文件，返回错误。
此时的解决方法是考虑直接把lib目录添加到sys.path中，类似如下：

	import sys, os
	basepath=os.path.abspath(os.path.dirname(sys.path[0]))
    sys.path.append(os.path.join(basepath, 'lib'))
	
3.sys和os.的path函数
sys模块相关函数主要是与python解释器交互，os模块相关函数主要是与操作系统交互。os本身包含很多功能，比如调用shell脚本（os.system），这里只限制在os.path常用几个函数的功能上进行说明。
	
分组来看：	
(1).os.path.abspath返回当前路径的绝对路径；os.path.relpath返回当前路径的相对路径。
(2).os.path.split(a)将当前路径拆分为两个部分，前面的是文件的路径，后面的是文件名（也可能是目录和子目录）；os.path.join正相反；os.path.dirname返回的是其中前面的部分，os.path.basename返回的是其中后面的部分。
(3).os.path.isfile判断输入字符串是否是一个文件；os.path.isdir判断输入字符串是否是一个目录；os.path.exists判断路径是否存在
(4).os.path.getmtime获取文件最后一次更新的时间；os.path.getatime获取文件最后一次访问的时间。
