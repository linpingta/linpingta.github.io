---
layout: post
title: "flake_check"
date: 2016-01-01 20:03:55 +0800
comments: true
categories: [Python]
---

最近在项目组打算组织了一次规模较大的code review，也是借这个机会检查自己程序上的问题。当然程序会有不同层次上的问题，比如模块的解耦，功能的设计，再到测试用例的添加和
代码书写的规范。今天主要要介绍的就是一款代码书写规范方面的模块: [flake8](http://flake8.readthedocs.org/en/latest/index.html)

flake8包含了下面三种模块：
	
	PyFlakes
	pep8
	Net Batchelder's McCabe script
	
PyFlakes中包含了一些code style方面的规范，pep8是python官方的[编程规范](https://www.python.org/dev/peps/pep-0008/)，McCabe是一个代码复杂度方面的检查脚本

flake8的安装：

    pip install flake8 / easy_install flake8
	
flake8对项目/文件的检查
	假设有module命名为A，其中包含a.py文件，那么可以在项目级别做代码规范检查：
	
	flake A
	
	也可以在文件级别做代码规范检查：
	
	flake8 A/a.py

flake8的配置
	flake8支持两种级别的设置：
	
	1. Global级别： 修改 ~/.config/flake8 
	2. Project级别：在Project的根目录增加一项tox.ini文件，在其中增加配置
	
		[flake8]
		max-line-length = 200 # 修改默认的单行字数长度
		
	（较为常用，因为通常由于权限原因，我们并不能做Global级别的设置，同时Project级别的设置也是可以满足要求的）

	通过这些设置，可以避免一些不需要的报警，比如单行字数限制为80通常会引起较多的code报警
	
flake8报警类型：
	
	以E**/W**开头：pep8的error和warning
	以F**开头：PyFlakes的代码检查
		code	sample message
		F401	module imported but unused
		F402	import module from line N shadowed by loop variable
		F403	‘from module import *’ used; unable to detect undefined names
		F404	future import(s) name after other statements
			 
		F811	redefinition of unused name from line N
		F812	list comprehension redefines name from line N
		F821	undefined name name
		F822	undefined name name in __all__
		F823	local variable name ... referenced before assignment
		F831	duplicate argument name in function definition
		F841	local variable name is assigned to but never used
	以C9**开头：McCabe的代码复杂度检查
	以N8**开头：命名检查

flake8与Git关联
	在.git/hooks/pre-commit中添加设置
	
	import sys
	from flake8.hooks import git_hook
	
一些常见的代码格式问题：
	
	单行长度
	class / def 间的空行数
	[l.append(x) for x in x_list] 中的空格数
	单行中加注释与代码间的空格间距
	。。。
	
以上只是一些简单的应用，与flake8有相似功能的模块还有更出名的[pylint](http://www.pylint.org/)，flake8本身的功能并不复杂，但对于规范代码书写上的要求，的确需要更多
提高，这方面我还做得不足，需要更加注意和重视。
