---
layout: post
title: "Web数据查询"
date: 2014-07-08 22:21:30 +0800
comments: true
categories: [Github Project]
---

前期处理的所有数据最终需要反映在数据应用中。我用Django简单搭建了一个Web数据查询页面，支持用户根据
	
	地区（城区、区域、小区）
	时间范围
	租金限制
	楼层限制
	房屋限制
	朝向限制	
	
进行查询。

![S1](/images/2014/07/system1.png)

![S7](/images/2014/07/system7.png)

除时间外的选择均基于jquery的select2插件，时间控件基于jQDateRangeSlider插件实现，主要采用Ajax做前后台的数据交互，整个交互分为“初始化”和“查询”两部分，“初始化”负责加载selector部分信息（区域和小区的信息是根据城区的选择结果通过Ajax加载的），“查询”部分执行selector选择后的SQL，并返回指定格式的json数据

![S8](/images/2014/07/system11.png)	

![S8](/images/2014/07/system12.png)	

查询结果包括6个页面：（有一个未实现，本身想做饼状图）

	1. 报表模块 ：文字角度的数据展示

![S2](/images/2014/07/system2.png)	

	2. 地图模块 ：显示当前查询信息的地理位置

![S8](/images/2014/07/system8.png)	

	3. heatmap : 以地区房屋的size和count为变量显示

![S4](/images/2014/07/system4.png)	

	4. 直方图：显示不同区域的价格对比

![S5](/images/2014/07/system5.png)	
	
	5. 时间序列： 显示不同区域的价格随时间的变化

![S6](/images/2014/07/system6.png)	
	
整体效果：

![S9](/images/2014/07/system9.png)	