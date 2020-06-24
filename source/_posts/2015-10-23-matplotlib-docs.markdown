---
layout: post
title: "matplotlib-docs"
date: 2015-10-23 09:42:09 +0800
comments: true
categories: [Python,Matplotlib] 
keywords: 
description: 
---


matplotlib.pyplot : 提供类似matlab中的绘制功能

	import matplotlib.pyplot as plt

常用函数：
1.plt.plot

plt.plot(y) ：x取值为[0, len(y)-1]
plt.plot(x,y) ：x,y以pair的形式绘制
plt.plot(x,y,format)：format包含color和style两部分，格式是color string 
concatenated with style string, 比如'ro'表示红色的点阵
plt.plot(x1,y1,x2,y2...)：同时绘制多条曲线
另外还包括很多指定线属性的方法，具体见[官方文档](http://matplotlib.org/devdocs/api/pyplot_api.html#matplotlib.pyplot.plot)。

2.plt.axis & plt.figure
绘图时，matplotlib有当前axis和当前figure的概念，通过gca()和gcf()获取它们，通过cla()和clf()清除它们。
plt.figure(n)指定当前绘制第几幅图，默认情况下，matplotlib会隐含的调用plt.figure(1)来开始绘制，因此有时候并不需要显式调用plt.figure。

	plt.figure(1) # 绘制第一幅图
plt.subplot(rows, cols, cur\_fig\_num)用来切割当前图完成子图绘制，相关概念和matlab一致。

	plt.subplot(211) # 切割两幅图中上面的一幅
	plt.subplot(212) # 切割两幅图中下面的一幅
plt.axis([x\_min, x\_max, y\_min, y\_max])指定绘图的取值范围

	plt.axis([40, 160, 0, 0.03])
	
3.plt.text , title, xlabel, ylabel
指定文字位置，其中text可以通过指定x和y坐标（即x,y列表中的某一对取值）来指定图中未知
title, xlabel, ylabel在图中均是相对固定的位置。
可以进一步指定文字的颜色和大小：

	t = plt.xlabel('my data', fontsize=14, color='red')
plt.annotate用于标注图片中特定点，它需要指定待标志的位置，以及标志文字出现的位置。

	plt.annotate('local max', xy=(2, 1), xytext=(3, 1.5),
            arrowprops=dict(facecolor='black', shrink=0.05),
            )

4.plt.tight_layout
自动调整布局，避免字符被截断或者出现重合，在[这里](http://matplotlib.org/devdocs/users/tight_layout_guide.html)

5.plt.legend
图例，添加对绘制对象的描述。
可以单独调用，也可以通过label属性和绘制对象绑定。

	line, = ax.plot([1, 2, 3], label='Inline label')
	# Overwrite the label by calling the method.
	line.set_label('Label via method')
	ax.legend()
6.plt.setp
设置某个或某组artist对象的属性，例如：

	lt.setp(labels, rotation=30, fontsize=10)

在matplotlib中API分为三个层级：

	1.FigureCanvas:决定绘制的地方
	2.Renders：决定绘制的方式
	3.Artist：决定如何调用1和2
通常情况下，matplotlib程序中操作的对象都属于artist，它具体又可以划分为两个类别：primitives和containers，前者是绘制对象，后者是绘制的容器，举个例子，前者是line，后者是figure。因此，最终的绘制，可以视为在对象中添加对象的过程，例如：

	fig = plt.figure() # figure对象
	ax1 = fig.add_suplot(211) # 添加第一个subplot对象
	ax2 = fig.add_axis([1,2,3,4]) # 添加第二个axix对象
	print fig.axes # ax1和ax2都是figure下的对象
上例中的ax1和ax2都属于Axes（不是axis）类，这是matplotlib中最常用的一个类，它自带了如plot()，text()等函数，用于绘制Line, Rectangle等promitive对象。
每次绘制，都是创建一个artist对象，然后把它放置到某个container的过程。

与pandas集成

通常使用Python进行数据处理后，我们可能会得到一个pandas.Series或pandas.DataFrame对象。pandas在内部与matplotlib进行了集成，使得基于数据结果的绘制非常方便。
文档见[这里](http://pandas.pydata.org/pandas-docs/version/0.18.1/visualization.html)

常见问题
1.在无界面的Linux引入报错无模块tkinter：

	import matplotlib
出现错误：

	import _tkinter # If this fails your Python may not be configured for Tk
	ImportError: No module named _tkinter
解决方法：

	import matplotlib
	matplotlib.use('Agg') # 强制使用agg作为backend
原因：
		在官网的[这里](http://matplotlib.org/faq/howto_faq.html#matplotlib-in-a-web-application-server)，对于没有界面的系统，你必须显式指定绘制哪种类型的图片(PNG, PDF, SVG)。
		