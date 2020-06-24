---
layout: post
title: "Motion Tracking in Image Sequence"
date: 2014-07-26 23:28:42 +0800
comments: true
categories: [Image]
---

Motion Trackng算法研究
================

* 来源
视频跟踪问题的应用十分广泛，比如军事跟踪或者交通识别。这次研究的背景问题来自一个具体的
应用需求: 对于马路上固定摄像头下统计彩色图像中的车流量，并区分其中的客车和公交车。关于
这部分的应用设计打算另文阐述，本文主要阐述在调研此问题解决方案时接触到的一个很好的多目标视频跟踪框架[Motion Tracking in Image Sequences](http://www.stats.ox.ac.uk/~wauthier/tracker/)。

* 多目标视频跟踪框架
    图像检测和分割针对单帧图像，而跟踪问题涉及多帧图像间的交互。然而在多帧图像中，既存在新目标的进入（目标提取和初始化）也存在旧目标的运动（目标状态变换），必须对目标进行区分。相对于单目标跟踪问题中“检测+(预测)+跟踪”，多目标跟踪包括“检测+预测+匹配+跟踪”。

Motion Tracking in Image Sequences包括以下结构：来自[这篇文章](http://www.robots.ox.ac.uk/~cvrg/hilary2002/vsam-pami-tracking.pdf)
    
    1. 目标提取
        Background model estimation + Connected components
    2. 目标预测
        Kalman filter
    3. 多目标匹配
        Munkers Algorithm (匈牙利算法) 
    
* 目标提取 (Object extraction)
    
    这一步的主要目的是从当前帧中提取有效的前景区域并对其根据连通域分割描述。

    * Background model estimation
    这种方法不同于“背景相减法”，它的强大之处在于可以适应背景自身的变化。“背景相减法”必须包括一个标准背景 (standard_background_image)，那么它的选择主要有两种策略：(设当前帧为ImmageN)
        1. 用第一帧做背景
        2. 用第N-K帧做背景
    前者对于环境中背景本身的光照变化没有抵御能力，后者虽然有一定的抵御能力，但是K的选择基本是经验性的，假如很不幸恰好选择的第N-k帧包括一定的噪声（可能哪帧图片曝光不好），那么之后的跟踪基本一定失败，在背景选择上，上述两种方法都是hard-code，即背景来自且只来自某一个时间的图像。
    相比于“背景相减法”的hard-code，Background model estimation更像是soft-code，如果一个目标持续的存在于原先的背景中（比如开入场景然后停下的车），经过一段时间的学习，它会转换为背景。作者在[文中](http://www.stats.ox.ac.uk/~wauthier/tracker/tracker-synopsis.pdf)提到的一个实验是对于固定摄像头下一天出现的目标识别，白天和夜晚光线的变化可以很好的被此模型学习。
    它的具体做法如下：
    
        以像素为单位进行研究（如果是彩色图像就是RGB三通道），假设每个像素点在t时刻的颜色为 Xt = I(x0,y0,t)，那么从1~t内像素的颜色序列为 {X1,...,Xt}，Xt被视为不同μ和Σ下混合高斯的叠加：
    P(Xt) = Σ w(t,i) * η(Xt,μi,Σi)
    （具体的公式神马的还是直接看[原文](http://www.robots.ox.ac.uk/~cvrg/hilary2002/vsam-pami-tracking.pdf)，打字有些不方便）
    之后的操作分为预测和改进（和EM很像），判断当前Xt和哪个Gaussian比较吻合（小于2.5Σ），找到最吻合的Gaussian，标记相应的Index，如果都不满足要求，则当前位置属于foreground,用当前Xt计算的μ和Σ去替换原先与其最接近的Gaussian（给一个学习的机会）。
    对于未选定的Index，不更新它的μ和Σ。对于选定的Index(即最接近描述对象的原先背景区域)，更新它的μ，Σ以及权重w。
    具体公式之类的还是需要看下原文，我对这种做法的理解是，如果当前像素值很像背景，那么首先认定它为背景，然后根据它的值对于背景进行更新，因此对于细微的背景变化，它能够达到调整背景区域的目的。而如果当前像素值不是背景（不属于所有的Gaussian），那么认定它为前景，同时给它一个更新为背景的机会（也并不是直接替换，因为相应的Gaussian初始权重很小，需要不断学习匹配才能累加它的权重）
    
    * Connected Components
    这部分倒没有太多好讲的，主要就是对于当前帧中前景区域的划分和描述（object_size, object_pos, object_boundary, label），产生的有效内容（比如object_size>N）被称为observations，即当前帧可能的object区域。
    
* 目标预测

    目标跟踪相比单帧的图像分割，最大的一个区域在于利用帧间信息。这里采用[Kalman filter](http://blog.csdn.net/lanbing510/article/details/8828109)对已有的object的状态进行预测。这句话里，“已有”指的是根据以前帧中observation产生的object，因此observation是有转换为object的能力的，但显然要区分哪些是新的对象。而“状态”包括object的pos_x,pos_y,vel_x,vel_y和obj_size。根据历史信息预测的object和当前帧分割出的observation，是下一步匹配的基础。

* 多目标匹配
    
    * [匈牙利算法](http://www.renfei.org/blog/bipartite-matching.html)

    匈牙利算法的目的是解决二分图的最大匹配，所谓二分图，就是图中没有奇数条边的环的图。最大匹配是指在图中被匹配的边数最多。它的核心是每次遇到从非匹配点到非匹配点的交错路径时，就进行一次翻转，因为每次翻转都可以增多匹配个数。
    在此问题中object和observation可以认为是二分图的两侧，具体的工作模式我还需要再做一些理解。