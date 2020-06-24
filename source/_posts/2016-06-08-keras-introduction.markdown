---
layout: post
title: "Keras深度学习框架(介绍)"
date: 2016-06-08 22:14:42 +0800
comments: true
categories: [Python,Machine Learning] 
keywords: keras, deep learning
description: keras, deep learning
---

keras是一个高层次的深度学习模块，底层依赖Theano(默认)或TensorFlow调用，封装了Theano中的功能，简化深度学习相关功能的调用。

Sequential : 用于组合一系列layers，是keras最基础的类。
最常用的网络定义模式如下：

	models = Sequential()
	models.add(Dense(32, input_dim=784)) # 添加model1
	models.add(Activation('relu')) # 添加model2
	# 添加其它model
	...
	# 定义模型
	models.compile(optimizer='sgd',loss='categorical_crossentropy', metric=['accuracy'])

	# 训练模型
	models.fit(train_x, train_y, nb_epoch, batch_size)
	
	# 输出预测
	y_pred = models.predict_class(test_x)
通过非常清晰简单的方式，我们就可以定义一个深度学习网络(models.add)，定义它的优化方式(models.compile)，然后训练模型(models.fit)，最后应用预测(models.predict)。问题的核心仍然在网络构建本身，但通过预定义的model（例如Dense, Activation）使得网络定义本身也得到了简化。

下面本文介绍下keras主要的应用模块。

1.layer本身相关部分

所有layer都有一些公有函数：
(1) get_weights() 返回层中神经元的权值，numpy.array
(2) set_weights() 设置层中神经元的权值
(3) get_config() 返回layer的配置

如果layer是非共享的，那么可以通过

	layer.input
	layer.output
	layer.input_shape
	layer.output_shape
获取层的输入张量，输出张量，输入形状，输出形状。
如果layer是共享的，那么通过 get_xx_at(node_index) 获取指定node的信息。

(1)Dense：较常用的一种Neutral Network layer。

定义

	keras.layers.core.Dense(output\_dim, 
	init='glorot\_uniform', activation='linear', weights=None ...)
	
	主要参数：
	output_dim：输出向量
	input_dim：输入向量
	activation: 定义传递函数，如果不指定，默认为linear, f(x)=x

调用实例，输入节点16个，输出节点32个：

	model = Sequential()
	model.add(Dense(32, input_dim=16))

(2)Activation: 定义激活函数activation：

	keras.layers.core.Activation(activation)
输入层和输出层维度保持一致。

Dropout: 在模型训练的时候使得部分节点的权重不更新，包含参数p，取值范围是[0,1]，指定一定比例的训练样本不用于节点更新，避免过拟合。

	keras.layers.core.Dropout(p)

	主要参数：
	p：[0, 1], 指定一定比例的样本不用于更新

其它相关函数可以参考[这里](http://keras.io/layers/core/)。

2.卷积相关layer定义
(1)Convolution1(/2/3)D ：卷积操作，分别针对1维、2维和3维的输入操作。

	keras.layers.convolutional.Convolution1D(nb_filter, filter_length, init='uniform', activation='linear', weights=None, border_mode='valid', subsample_length=1, W_regularizer=None, b_regularizer=None, activity_regularizer=None, W_constraint=None, b_constraint=None, bias=True, input_dim=None, input_length=None)

	主要参数：
	nb_filter：指定卷积核的个数（等价于输出的维度）
	卷积核的形状：
		一维是长度：filter_length
		二维是形状：nb_row, nb_col
		三维是形状：kernel_dim1/2/3
	输入形状：
		一维例如：（10,30），前者是样本数量，后者是每个样本的维度
		二维例如：（3,256,256），前者是样本数量，后面两项是图像的长宽

(2)MaxPooling(1/2/3)D / AveragePooling(1/2/3)D：主要解释下pooling，它有点像是降采样，把一定区域内的特征按max或average的方法变为一个特征，用于降低处理的数据量。

3.optimizer

定义keras模型的优化方法，keras支持两种类型的输入，
(1) 指定optimizer类的实例，如：

	model = Sequential()
	#...
	sgd = SGD(lr=0.1, decay=1e-6, momentum=0.9, nesterov=True)
	model.compile(loss='mean_square_error', optimizer=sgd)
	
(2) 指定optimizer类的名称，如：
	
	model = Sequential()
	model.compile(loss='mean_square_error', optimizer='sgd')

可选的optimizer方法包括SGD, RMSprop，Adagrad，Adadelta等，这些方法主要的区别在于learning rate的计算，learning rate作为超参数的一种，在早期的神经网络中并不能通过网络本身学习，而是通过使用者的经验（调参）决定，因此learning rate的计算方法本身也是深度学习重要的研究方向之一。简单的理解（或者说以我的理解），learning rate可以采取固定值（经验参数，策略一），也可以采用前期跨度大，后期跨度小（策略二），类似的不同策略衍生出不同的optimizer，更详细的科学解释可见[这里](https://www.quora.com/What-are-differences-between-update-rules-like-AdaDelta-RMSProp-AdaGrad-and-AdaM)。

4.objective

定义keras模型的损失函数。同optimizer，keras也支持两种类型的输入：
(1) objective的名称，常用的如：

	mean_square_error/mse
	binary_crossentropy
	categorical_crossentropy
(2) Theano/TensorFlow的函数，类似[如下](https://github.com/fchollet/keras/blob/master/keras/objectives.py)：
	
	def mean_squared_error(y_true, y_pred):
	    return K.mean(K.square(y_pred - y_true), axis=-1)

5.activation

神经网络中的激活函数（=转移函数），常用的比如：

	model = Sequential()
	model.add(Dense(64, activation='tanh'))
添加了tanh作为激活函数。这里同样可以传入函数对象：

	def tanh(x):
	    return K.tanh(x)

	model.add(Dense(64, activation=tanh))
	model.add(Activation(tanh))
	
6.initializations

网络层的初始权值，不同层可以定义不同的权值分布，例如：

	model.add(Dense(64, init='uniform'))
定义一个包含64个节点，初始权值一致的网络层。

7.compile函数
objective和optimizer是compile函数必须的两项输入参数，例如：

	model = Sequential()
	model.compile(loss='mean_square_error', optimizer='sgd')