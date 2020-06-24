---
layout: post
title: "数据表设计"
date: 2014-07-07 22:51:43 +0800
comments: true
categories: [Github Project]
---

数据表设计
----

对于租房问题，我想最基本的分析需求包括：

    (1) 查询某地区(region)某面积范围下(size)的租金价格： 此点应该是用户最开始关心的内容
    (2) 比较两地区的租金价格
        select region_id, size, price from table where region_id = xxx and size  between x1 and x2;     
    (3) 某地区的租金随时间的变化关系 （时间维度应用）     
    (4) 房子尺寸与租金的关系，房屋朝向与租金的关系 （产品维度应用）

建立支持分析的数据表。对于租房信息的分析，非常适合数据仓库系统的特征

    面向主题：租房
    集成：这点主要指异构数据库的数据源分析，还是推荐microstrategy的产品吧
    非更新的：中介通常会发布一个新的房源信息（新的url）而非更新已有url中数据
    时间累积的：租房系统中的数据包含同一地区不同时间的发布信息

按照kimball的理论，采用非规范化的、维度的描述分析问题。

但另外一方面，对于获取的数据（网页爬取数据），通常具有较大的数据量，非规范化需要更多的存储空间（重复数据）
因此我的设计决定对于爬取数据（ODS层）和分析数据（WH层）采用不同的数据存储方式：
    
    在ODS层采用规范化的数据范式
    在WH层采用半规范化的存储方式

但这里这样做还有另外两个原因（考虑小本经营，搞两套表然后去转还是要花一些时间的）：

    1. 学习Kettle，或者说ETL工具的使用
    2. 实践microstrategy的产品理念 （半规范化）

那么对于基本分析中的需求：
    (1) 地理位置信息
    (2) 小区信息
    (3) 时间信息
    (4) 房屋信息

做一些简单扩展说明：

    (1) 地理位置信息 ，例如 北京 - 海淀 - 西二旗 - 当代xx家园
    地理信息是一个自然的继承关系          
     LU_CITY :  CITY_ID, CITY_NAME
     LU_DISTRICT : CITY_ID, DISTRICT_ID, DISTRICT_NAME
     LU_LOCAL : CITY_ID, DISTRICT_ID, LOCAL_ID, LOCAL_NAME
     LU_DETAIL_LOCAL : CITY_ID, DISTRICT_ID, LOCAL_ID, DETAIL_LOCAL_ID, DETAIL_LOCAL_NAME

    (2) 小区信息：绿化率，供暖方式，开发商，开发时间， 具体地址
     LU_DETAIL_LOCAL:  再添加 GREEN_RATE, WARM_TYPE, OPEN_TIME, OPEN_COMPANY_ID, SERVICE_COMPANY_ID
     LU_OPEN_COMPANY:  存储比如 xxx开发商信息
     LU_SERVICE_COMPANY: 存储比如 xxx物业信息

    中介信息：姓名 联系方式 简介 (与房屋是多对多关系)
     LU_AGENT: name, tel, introduction
    
    教育信息 （尽管对租房用处不大，比如是否学区）          
    EDUCATION

    (3) 时间信息：
    在爬取层，时间信息并不需要过多的扩展，只需要存储房源的发布时间和爬取时间（发布时间
    用于一定时间范围内的分析，爬取时间用于比较信息准确性），所以这部分在WH转换层做说明
    在爬取层我只是把它作为最低level的事实表的一个字段做保存
    
    (4) 房屋信息
              
        名称   租金  户型  面积  朝向 楼层  楼龄 装修时间 信息来源  付款方式
          HOUSE_RENT_INFO ：
               HOUSE_ID HOUSE_NAME ROOM_TYPE_ID STAIR_TYPE_ID DIRECTION_TYPE INFO_SOURCE_ID PAY_TYPE_ID
               DECORATE_TIME 
               RENT_M   SIZE_M          

          户型信息表
          LU_ROOM_TYPE:  ROOM_TYPE_ID, ROOM_NUM, SPACE_NUM (几室几厅)
          
          楼层信息表
          LU_STAIR_TYPE:  STAIR_TYPE_ID, STAIR_NUM, MOST_STAIR_NUM (6/12)

          信息来源表
          LU_INFO_SOURCE:  INFO_SOURCE_ID, INFO_SOURCE_NAME  (链家)

          付款方式表
          LU_PAY_TYPE: PAY_TYPE_ID, PAY_NUM, LOAN_NUM (押一付三)

从事后的完成情况看，(1),(3),(4)与应用结合比较紧密，(2)的应用做的并不好。

一些相关的设计问题：

     1. 比如对于绿化率，有些网站会写明百分比，有些网站则是用描述性，（通常是高，好，或者无描述），实际描述的是同一个概念，但是数据的整合遇到一定困难，这应该是ETL要处理的一部分内容
     比如对于装修，有些网站会写2008年，有些会写五年以内，实际上是一回事，或者比如6楼中部，和3楼实际上是一回事，但是在处理前需要统一
          或许这就是为什么数据库可以存储分类数据，而数据仓库中的数据必须经过汇总处理
     
     2. 对于频繁查询的东西是否需要固化信息？
     比如户型，单独是可以存在的，也可以存在房屋列中 -- 最终决定作为ID单独存在

     3. 如何在数据库里实现同步插入呢
          比如对下面三个表，在选取数据时能满足join操作要求，但在插入数据时候如何方便建表？
          
          应该写类似如下的存储过程
          
          if city.name doesn't exist, insert city.name, auto_increment city.id,  record city.id;
          else select city.id from LU_CITY where city.name = input_name

     4. 如何确定哪些属性是表固有，哪些属性应该分不同表 -- 
     关于建模的信息我拟参考ER模型相关知识
          ER： 实体集，关系集，属性
          是用实体集还是属性，如果是一对一关系，那么用属性，否则用实体集
               比如身份证，年龄，作为人的属性
          其实是实体集还是属性的分别是很模糊的，如果比较清楚的区分自然而然可以区分，而不清楚的应该本身就允许多种设计关系的存在，
          户型可以作为房屋的属性，也可以作为单独的实体然后通过ID联系     
         
     5. 数据表的设计可能会随着业务的变化而变化
          比如 学区房 信息， 当前是不需要的，但是如果是买房业务，学区房应该作为一个单独的条目存在

     6. 比如题目中的信息是很难挖掘的，因为没有规范的格式，但同时也是很重要的，因为它基本说明房屋attractive的地方
     例如：
          随机抽取不同网站的题目，暂时只保存不处理

具体建模

![SOFTWARE_STRUCTURE](/images/2014/07/table_design.png)
	