---
layout: post
title: "google server to server 广告对接"
date: 2016-09-21 17:28:08 +0800
comments: true
categories: [Advertise] 
keywords: 
description: 
---

一. 为什么进行广告对接
互联网广告相比传统广告的优势之一，就是它的效果可以被监控。只有通过监控，如Google这样的大型广告服务公司才能了解广告投放效果，并且进一步按照用户的需求（主要是成本）进行优化。

二. 广告对接形式及问题
最常用的对接形式是嵌入SDK，比如Google和Facebook都有自己的SDK，在应用中添加后，当指定事件发生时，SDK会向服务器汇报事件，作为统计的依据。
另外还有一些第三方的数据监控公司，比如TalkingData, Appflyers，提供数据监控服务。
嵌入SDK主要有三方面的问题：
1. SDK 大小。 应用大小本身对用户体验有直接影响，SDK作为一部分代码添加到应用中，会直接增加应用大小，这是应用开发者或者广告主最为担心的。另一方面，如果多家广告公司（Facebook, Google, 第三方）都有自己的SDK，而应用不得不将它们都添加到应用本身的话，肯定是一个不小的量级，尤其对非游戏类app，这点是难以接受的。
2. 效率。类似第一点，SDK在事件汇报时可能会占用资源，影响用户体验
3. 安全。这点也比较明显，毕竟在自己的产品里嵌入一段不懂的代码，很可能担心它有其他影响

由于上面的问题存在，现在有了一种通过server to server(s2s)方式完成事件汇报的方法。

三. s2s 形式

如名称所示，事件汇报不再是通过SDK由应用（client）直接汇报给广告服务公司（例如Google），而是由广告主自己监控事件后，发送给广告服务公司。
需要说明的是，这里的事件指的是如应用激活或者应用内行为等后续事件，不包括广告点击和展现事件。对于点击展现事件，广告公司可以直接监控到它们。
因此s2s的事件汇报，按另外一种说法，可以称为激活核对。广告公司有所有的点击事件，广告主有所有的激活事件，它们都可以对应到设备ID（对苹果是idfa，对安卓是aaid），下面要做的事就是确定哪些激活来自哪些广告。

假设广告主为甲方，广告公司为乙方，那么按汇报方向划分，s2s可以分为两种形式：
1. 甲->乙: 广告主把自己的所有激活设备ID告知广告公司，广告公司去做点击匹配，返回广告主其中哪些设备ID的激活是来自自己的广告
2. 乙->甲: 广告公司把自己的广告点击ID告知广告主，由广告主匹配激活，返回广告公司哪些激活是来自广告
通常一个设备激活可能会有多个点击存在时，按最后一次点击的广告确认为激活来源广告。

四. Google s2s方式
首先，Google s2s的方式只支持上述“甲->乙”，即广告主汇报事件给Google的形式。
具体而言，向Google的指定网址发送一个Get请求，格式如下：

	GET https://www.googleleadservices.com/pagead/conversion/conversion_id/?label=conversion_label&bundleid=xx&idtype=xx&rdid=xx

具体解释其中的参数：

	conversion_id和conversion_label：广告主在Adwords上面创建账户时，会填写这两部分内容，这里的请求中要对应填写，用于确定广告信息（这里我不太确定是否是每个campaign有不同的conversion_id，但我感觉应该是一个广告主身份的确认标志，而不是campaign级别的确认）

	bundleid：对于应用而言，就是包名，比如com.example

	idtype：确定应用类型，只有2个值可选，分别是idfa和aaid，因为ios和android的广告汇报方式一致，所以在这里告知Google应用的类型

	rdid：设备的IDFA（对安卓就是aaid），这里填的是实际值

这里idfa和aaid都不要加密。
其它可选的输入参数还包括：

	appversion：告知app版本信息
	value：广告主定义的价值
	currency：价值对应的货币单位

对于上述请求，Google服务器会返回相关信息到广告主指定的网址：

	https://广告主网址?advertising_id=ad_id ...

其中广告主网址需要在Google Adwords中配置，配置后服务器才能将信息发送到网址，返回的参数（当然你也可以理解为对广告主服务器的请求）包括：

	ad_id：未加密的android设备ID，或者加密的ios md5
	click_url：广告点击网址，包含点击的所有信息
	click_ts：广告点击时间
	campaign_id, video_id：这里campaign并不是广告活动的ID，而是特指youtube的参数，video也是youtube特有的参数

在向Google服务器请求时，返回码可能有三种状态：

	1.200：表示请求格式正确，但激活并不是来自Google广告
	2.302：表示请求格式正确，且激活来自Google广告
	3.400：表示请求格式不正确

五. 与广点通异同
这里有ppt就很清楚了，可惜现在还没有，只能大致说下：

	广点通支持 甲->乙 和 乙->甲 两种对接方式，Google现在只支持一种
	其它区别主要是字段含义方面，但大致信息是一致的
	
六. 测试
后续结束时有一个实操测试，测试指定IDFA是否来自Google服务器，但感觉还是骗人的成分多些，实际对接要复杂不少，附Python代码如下：

    def get_idfa(idfa):
        try:
            url = "https://test-1470278950694.appspot.com"
            conversion_label = 'abCDEFG12hlJk3Lm4nO'
            conversion_id = ''
            bundleid = 'com.example'
            params = {
                'rdid': idfa,
                'label': conversion_label,
                'bundleid': bundleid,
                'idtype': 'idfa',
            }
            res = requests.get(url, params=params)
            print res.status_code
            return res.status_code
        except Exception as e:
            print 'error', e

