---
layout: post
title: "Advertising Class"
date: 2014-07-20 18:56:53 +0800
comments: true
categories: [Advertise]
---

Computing Advertisement

Find the best ad for the user in the context.
(Find the "best match" between a given user in a given context and a suitable advertisement)

第一章 基本概念简介

广告类型分类

* 搜索广告： 百度等等
* 展示广告：
    * Rich media : 指的是广告的形式以html/flash展现，强调交互性
    * Digital video : 指的是视频广告，比如youku播放视频前的广告内容
         
            In-stream : 指的是通过播放器展现的广告
            
                linear video ad : 在视频的开始/结束时播放的
                non-linear video ad : 在视频中间（比如电视剧之间的插播，或者延迟等待）播放
    * Banner ad : 边栏广告，最典型的是横幅广告，比如新浪首页广告，通常是品牌广告
    * Sponsorship ad : 赞助广告
* classfied 广告：传统报纸中夹页位置出现的广告
* lead generation : 根据用户做推送
* email

广告主分类：
指的是行业分类，比如零售业，快消业

付费方式：
CPM：每次展示， 比如一般侧边栏广告或者图像类广告采用此类
CPC: 每次点击， 比如一般文字类广告
CPA：每次效果

CTR: click through rate

Textual ad : 比如搜索广告，或者上下文广告
    Textual ads are heavily related to Search and IR

广告的均衡
1. 广告主：要的是ROI
2. 用户：要的是相关性
3. 媒体：要的是收入
4. 广告平台：要的是收入和成长
Ad selection : optimize for a goal that balances the utlites of the four participants.

海量的用户，广告和展示机会
快速的反应时间 （小于100ms）

解析用户特征（如果搜索广告，那么是强相关的关键词）
解析广告本身特征
IR (information retrieval) 对二者进行匹配

Search Engine 对于广告的应用

* Ad retrival: match to query/context
* Ordering the ads
* Pricing on a click-through

Ad retrival:
    广告主购买关键字，和query中提取的关键字做匹配

Ordering & pricing the ds:
    game theory : 每个参与者按照自己的策略进行出价
    auction theory : 每个人在auction market中的行为
    mechanism theory : design the rule of a game

auction theory:
    First-price sealed-bid : 所有人同时最高价取胜
    Second-price : 所有人同时，最高出价者以次高价取胜
    English auction : 常见的向高处叫价
    Dutch auction : 由高向低，直到第一个接受出价者即为成交价

第二章 

generize secondary price: 
    bid ranking : 任何第i个人都按第i+1个人的价格出价
    revenue ranking : 广告按出价和ctr排