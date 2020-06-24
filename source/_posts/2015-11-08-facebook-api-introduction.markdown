---
layout: post
title: "Facebook API (RESTful API)简介"
date: 2015-11-08 12:46:21 +0800
comments: true
categories: [Facebook] 
keywords: 
description: 
---

这里称之为简介实际也是勉强的，因为我所涉及的API主要只包含广告相关操作的API (称为Marketing API)和很小部分的用户行为API(称为Graph API，Marketing API是包含于Graph API中的一个小部分，Facebook有一个在西雅图的团队主要负责API相关的设计和各类语言支持。。因为见过他们的一些人。。扯得有点远)，但我更想借这个机会去了解API设计的规范，至少从我的角度看，Facebook API在使用上还是让人满意的，最后也会涉及一些Facebook 具体API调用方面的说明。

####RESTful 

1.RESTful与RESTful API：Representational State Transfer，表现层状态转化	

这实际是一个非常熟悉的概念，但如果突然被问到，却很难讲清楚。按我现在的理解，REST本身是一种架构规范，它指的是网络通信方式以 “资源（服务器对象）的表现形式上发生状态转换（增删改查操作）“的方式来实现，RESTful API指的是在设计规范上符合RSET架构的API。
RESTful API的一个重要特点是API本身没有动词，因为所有资源都是用名词表示的，通过GET/POST/DELETE/HEAD等http请求来模拟读写删等操作。因此RESTful API看起来非常清晰：

	GET graph.facebook.com/v2.4/me?field=id, name
获取me资源中id和name字段的值。

看起来是一件很简单的事情，不是么？但RESTful的重要价值就在于它提供了一种简单清晰的规范，让所有人都可以参照去做。或许你自己的网站也有一套url定义，但想想，如果你的网站不只是十几个，而是上百个页面的时候（从Facebook API对外提供的对象接口看，上百个应该只是保守说法），再或者说，当你的网站需要和开发者有交互时，这或许是最重要的原因，开发者通常来自世界，必须要有一套基础标准来规范通信方式，否则网站API升级一个版本导致以前的url全得重写，那开发者基本也会离你而去了。

2.RESTful API格式

格式即元素，每个RESTful API应该包含哪些元素：

	GET 
	https://graph.facebook.com/v2.4/ADSET_ID?fields=id,name&access_token=YOUR_ACCESS_TOKEN

	返回：

	{
	   "id": "ADSET_ID",
	   "name": "ADSET_NAME"
	}
* 协议：https
* 域名：graph.facebook.com，API通常有一个独立的域名
* 版本：v2.4，API在调用中通常要指定版本
* 对象(endpoint)：ADSET_ID，你要访问的资源对象
* 属性：fields，算是一个特殊的属性，它用于访问资源对象本身的属性
* access\_token也算是一个特殊属性，无论是Facebook还是Weixin的API，都需要权限，而权限信息就保持在access\_token中
* 操作类型：GET/POST是最主要的
* 返回值和状态码：status_code

关于RESTful的相关概念，应该还可以参考[知乎](http://www.zhihu.com/topic/19579308/top-answers)，但坦白说大部分内容我也没有看。

#### Facebook API

本文并无意（实际也没有能力）列举所有Facebook对象以及调用方法，但通过上文的说明，Facebook API是有固定的理解方式的：

	选择一个对象(endpoint)，选择它的属性(fields)，指定操作类型(GET/POST)，保证权限（access_token）
	然后，。。。API调用就完成了

具体的API调用文档可以打开https://developers.facebook.com/docs/，然后搜索要找的对象。

然而还有几点算是比较通用的（文档里常常要去关注的），我想有必要在这里列举下：

* changelog：在[这里](https://developers.facebook.com/docs/apps/changelog)，每个版本的API都有使用期限，如果超过会造成API调用失败，服务受到损失，通常情况下，开发不会总是去追最新的版本，因此必须注意版本的时效性，以及不同版本间对象的修改情况。（这里的一个例子是，Python Ads SDK中的对象命名在最近做了较大的修改，比如class AdGroup修改为class Ad）
* Explorer：在[这里](https://developers.facebook.com/tools/explorer/)，这是Facebook提供的一个web端调用页面（想想微信好像也有），使得用户不需要自己去拼http请求查看调用效果，在遇到新endpoint不熟悉其相关属性的时候非常方便，它本身也支持field字段的搜索，非常赞的功能。
* token：在[这里](https://developers.facebook.com/docs/facebook-login/access-tokens/)，token几乎是API调用的一个必须字段，理解token实际就是API是否有访问对象(endpoint)的权限，Facebook中token分为三类：App token/Page token/User token，我在工作中常用的是最后一项，相当于代表用户去操作广告
* batch_request：在[这里](https://developers.facebook.com/docs/marketing-api/batch-requests#adgroup)，批量操作在广告API中非常常见，比如用户在一次campaign创建时创建几百个adset，或者获取几百个adset的统计数据。因为Facebook本身对API的调用频率也有限制，超出后会遇到user call frequency limit的错误，因此很多广告操作必须要用批量接口实现。

暂时关于Facebook API能想到的东西就到这里，我这一年的使用感觉是，有问题看文档，Facebook作为世界最大的互联网公司，提供的文档本身是非常完善的，如果仍然有问题，可以关注Facebook上的一些技术小组，里面也有Facebook的工程师会回答问题。
