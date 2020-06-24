---
layout: post
title: "Weixin相关项目"
date: 2016-06-18 21:11:48 +0800
comments: true
categories: [Weixin,Github Project] 
keywords: 
description: 
---


1.[wx-api-alpha](https://github.com/linpingta/weixin-api-related/tree/master/wx-api-alpha)

	最开始的设想是想仿照Facebook Ads Python SDK封装一套weixin的API，但实际完成度很低，项目里主要包含的是一个token的自动获取（微信默认每一段时间token就会失效，因此每次调用前需要先确认token有效，这点可以通过token自动获取简化），以及基本requests对http调用的封装。

2.[dm-message-server](https://github.com/linpingta/weixin-api-related/tree/master/dm-message-server/weixin_server)

	微信对用户输入的交互示例
	以及与微信相关的任务管理，包括显示，编辑，创建和删除等

3.[weixin-monitor](https://github.com/linpingta/weixin-api-related/tree/master/weixin-monitor)

	微信报警模块的简单示例