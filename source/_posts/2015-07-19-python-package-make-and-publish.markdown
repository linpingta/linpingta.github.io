---
layout: post
title: "python package制作及类库发布"
date: 2015-07-19 10:11:32 +0800
comments: true
categories: [Python] 
keywords: 
description: 
---

很多人使用python，主要是被它非常丰富的库功能吸引。找到一个要做的事情，搜索对应的模块，然后用pip install或者easy_install就可以轻松的安装模块，然后使用。。。尤其对于一个之前一直工作在c++环境里，看到boost都兴奋很久的人而言，python的库功能无疑是我选择它重要的原因之一。

那么，是否可能自己制作自己的类库，然后发布给其它人使用呢？答案是肯定的，本文主要介绍如何制作一个自己的类库，供自己（在本地安装）或发布到pypi使用。

一. 制作类库

假设你的类库名称是first_test，那么类库应该满足如下格式：

	first_test/
	----setup.py
	----setup.cfg
	----README.md
	----first_test/
		---- __init__.py
		---- real_logic_file.py
其中setup.py和子文件夹first_test是必须的。

* 前者定义了类库生成的基础格式：

		from setuptools import setup
	
		setup(name='linpingta_first_test',
		        version='0.1',
		        description='linpingta first test for package',
		        author='linpingta',
		        author_email='linpingta@163.com',
		        url = 'https://github.com/linpingta',
		        packages=['linpingta_first_test'],
		        install_required=['time', 'datetime']
		        )
这里面参数中，值得注意的是install\_required：因为通常模块之间会有依赖，可能你的类库需要依赖一些基础类库，那么可以通过这个字段指定，这样pip知道在安装你的类库前先去安装你的依赖类库。name, version, author和author\_emaill应该是必须参数，其它参数应该是选填（不完全确认，可以使用时check下）

* 后者定义类库的实际内容

最简单的情况下，只需要在first\_test/\__init\__.py中定义逻辑，例如：

	def hello():
		return 'hello world'
但通常情况下，我们并不会把内容写在\__init\__文件中，而是使用单独的文件（例如real\_logic\_file.py），然后再在init中导入它，如下：

	__init__.py

	from .real_logic_file import hello

	real_logic_file.py
	
	def hello():
		return 'hello world'

ok，你的类库已经制作好了，如果要在本地发布（安装到site-packages或者dist-packages），那么可以执行 python setup.py install就可以了。

二. 发布到PyPI

或许你并不满足于类库只在本地使用，而是希望把它放到PyPI上面，供所有人通过pip安装使用。那么需要做下面几件事：

* 在PyPI上[注册账号](https://pypi.python.org/pypi?%3Aaction=register_form), 这里多说一句是，除了PyPI外，还有一个用于模块测试的[testpypi](https://testpypi.python.org/pypi)，用于模块正式发布前的测试，它是要单独注册账号的

* 填充setup.cfg和README.md
如果要使用markdown格式编辑README.md，那么在setup.cfg中添加如下一句：

		[metadata]
		description-file = README.md

* 创建.pypirc，用于保留给PyPI的注册信息，放在根目录上，内容如下：
	
		[distutils]
		index-servers =
		  pypi
		  pypitest
		
		[pypi]
		repository=https://pypi.python.org/pypi
		username=account_name
		password=account_password
		
		[pypitest]
		repository=https://testpypi.python.org/pypi
		username=account_name
		password=account_password

* 发布到PyPI，执行以下语句 （实际register, sdist和upload是三条独立的命令，可以分别执行）

		python setup.py register sdist upload

三. 参考资料及更多

上述的内容主要来自以下两篇文章：
1. https://python-packaging.readthedocs.io/en/latest/minimal.html
2. http://peterdowns.com/posts/first-time-with-pypi.html
感觉文章里写的不清楚的地方，请直接check它们:)
