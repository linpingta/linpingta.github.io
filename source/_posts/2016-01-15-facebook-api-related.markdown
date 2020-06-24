---
layout: post
title: "关于Facebook API可以考虑了解的几件事"
date: 2016-01-15 15:11:12 +0800
comments: true
categories: [Facebook] 
keywords: 
description: 
---


* Facebook API分为Graph API，Marketing API和Atlas API三种：

	* Graph API：是否还记得几年前Facebook推出的社交图谱，实际上它就是Graph API对应的逻辑概念。Graph API是另外两种API的基础，它实际是最主要也是最全面的Facebook对象获取方式。
	* Marketing API：与广告营销相关的API，比如作为FMP（Facebook Marketing Partner，像Domob），或者作为广告主调用
	* Altas API：实话说，我在今天之前都不知道这个API系列，看了下它的介绍，好像也是与应用和广告管理有关，但是里面的文档大多数又不可访问了，暂时打算忽略
	
* Facebook API开发必备：
	* 如果你是Python/PHP开发者（最近应该又造福了Java），可以在Github上搜索官方提供的SDK，比如[Python的](https://github.com/facebook/facebook-python-ads-sdk)。
	* 如果你用其它语言，那么可能你需要自己手写SDK，就需要涉及到构造Http请求，这时候推荐现在[Facebook Graph Explorer](https://developers.facebook.com/tools/explorer/145634995501895/)验证你的请求是否正常（容易的说官方的SDK也是拼接Http请求，基于requests库，但自己做起来还是很麻烦的，感谢官方SDK）
	
* Facebook API采用RESTful API格式设计，具体介绍可见我之前的[博客](http://linpingta.cn/blog/2015/11/08/facebook-api-introduction/)

* Graph API：每个Graph API包含三个部分：nodes, edges, fields，请求哪个对象（nodes）的哪个关联（edges）的哪个属性（fields），可见edges并不是必须的，完全可以通过nodes/fields=xxx的形式请求nodes本身的属性。举个例子， ad\_id/fields=name用于请求ad名称，ad\_id/creatives/fields=name用于请求ad下面创意的名称，在[这里](https://developers.facebook.com/docs/graph-api)

* Marketing API：实际也是Graph API的一个子集，在[这里](https://developers.facebook.com/docs/marketing-apis)

* Facebook API实例：[这里](https://www.facebookmarketingdevelopers.com/samples/)，看起来非常完全，不过是基于python的，我很遗憾我刚开始开发的时候没有发现这个东西（也可能是当时还没有），不过以后的开发，参考这里应该非常有帮助

* Facebook API文档：实际更广泛的说，应该叫Facebook developer文档，它本身服务的对象包括Facebook应用开发者，相关应用（比如Instagram, Oculus,）开发者，广告营销开发者，比如包含很多为移动开发者准备的IOS或则Android SDK，具体见[这里](https://developers.facebook.com/docs)。