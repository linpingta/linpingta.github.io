---
layout: post
title: "Kettle-first"
date: 2014-04-27 22:24:31 +0800
comments: true
categories: [Kettle,数据仓库]
---

目前在网上关于Kettle的中文材料比较少，也没有太成型的教程，之前在dataguru上面看到过有人开Kettle的课程，但需要付费，暂时工作里没有这方面的需求，因此这里结合看到的资料和实际的操作做一些简单的探究。Kettle包含的内容远超过本篇的内容，如果之后有机会使用我会继续补充。

Kettle (由于是Pentaho组件之一，官方名称为Pentaho Data Integration), 主要是一款ETL工作组件。它的[主要功能](http://wiki.pentaho.com/display/EAI/Pentaho+Data+Integration+%28Kettle%29+Tutorial)包括:
    1. 在不同数据库和应用间进行数据迁移
    2. 从数据库中向文本导出数据 （实际反向也已支持）
    3. 支持向数据库中加载多种数据源
    4. 数据清洗
    5. 应用集成

Kettle下载安装
Kettle是无需安装直接使用的，[下载地址](http://sourceforge.net/projects/pentaho/files/Data%20Integration/3.2.0-stable/),最新版本为3.2.0

下载完成后，可以在\pdi-ce-5.0.1.A-stable\data-integration路径下找到Spoon.bat运行，软件界面如下：

![TEST15](/images/2014/04/ETL_first_15.png)

Kettle把处理对象分为job和transformation两大类：
   
   1. Job负责工作流的控制，实际上我几乎没有接触
   2. Transformation负责数据的转换

因为我的应用均和Table相关，因此在创建一个Transformation的应用后，首先要做的是建立表输入和表输出。
选择 "核心对象"->"输入"("输出"), 可以看到Kettle支持的数据源很多，即官网说的支持多种数据源和数据库间的相互转换。

首先选择一项“表输入”，将其拖拽到右侧的界面。右侧出现以"表输入"为名称的item，双击对其进行编辑。

![TEST1](/images/2014/04/ETL_first_1.png)
![TEST2](/images/2014/04/ETL_first_2.png)

"步骤名称"建议根据应用需求改名；
“数据库连接”设置表关联到的数据库，点击"新建"，然后选择合适的数据库连接 （我在这里采用的是MySQL），连接后测试，如果显示下图，表示连接成功

![TEST3](/images/2014/04/ETL_first_3.png)

![TEST4](/images/2014/04/ETL_first_4.png)

连接成功后，还需要将当前item和数据库的一张表关联，点击“获取SQL查询语句”，找到需要的数据表，然后可以在"表输入"界面看到选择的数据。

![TEST5](/images/2014/04/ETL_first_5.png)

最简单的操作即将一张输入表和一张输出表发生关联，使得输入表的数据转到输出表中，这类操作通常应用于不同数据库间的数据转换，定义如下

![TEST6](/images/2014/04/ETL_first_6.png)

在"表输出"中可以显式的改变数据表字段的关联关系（我目前的操作均是如此），勾选"数据库字段"->"指定数据库字段"->"获取字段"

![TEST7](/images/2014/04/ETL_first_7.png)

在这里可以通过手动指定源与目标字段的映射关系来解决源表和目标表列不匹配的问题。

![TEST8](/images/2014/04/ETL_first_8.png)

在完成基本表关联的操作后，考虑到实际应用中大部分源表和目标表有着各种类型的不同，我们通过Kettle中的控件进行对应的处理

    "字段选择"控件
    指定哪些源字段会被输出到目标字段，同时允许改变字段的精度、类型等
    
	![TEST9](/images/2014/04/ETL_first_9.png)
	
    "过滤记录"控件
    比如源数据中有NULL类型，但是我们的输出中不需要这种类型，那么可以如下使用
	
	![TEST10](/images/2014/04/ETL_first_10.png)
	
另外还有三类常见的数据应用

1.  表连接
目标表中同时包含多个源表的数据

应用“记录集连接” (merge join)控件

在Kettle 连接选项卡里有多种连接方式

    Merge Join : 按照指定key连接数据，这是最普通的方式，数据需要按照join key排序
    Join Rows (cartesian product) : 笛卡尔乘积，连接后数据行数等于之前数据行数的乘积
    Database Join : 执行数据库连接
    Multiway Join : 多路连接
          
表合并要求合并前先将数据按join key进行排序，这点并不是非常方便，因为在SQL里面我们可以直接做join，而现在我们不得不增加一项额外步骤，最终连接效果如下图:

![TEST14](/images/2014/04/ETL_first_14.png)

2.  Javascript代码
“脚本”->"Javascript代码"，可以将Javascript脚本定义的变量转换为输出表的一个字段。

![TEST13](/images/2014/04/ETL_first_13.png)

3.  产生时间序列
对于数据仓库，时间纬度是非常重要的一部分，在一段时间内的year, quarter, month, day,以及派生出的 day_in_week, day_in_month, month_in_last_quarter等信息，可以通过Kettle的"计算器"中的原生函数产生。这部分内容较多，我之后单做一份进行说明，这里仅列出计算器的功能：

![TEST11](/images/2014/04/ETL_first_11.png)
![TEST12](/images/2014/04/ETL_first_12.png)