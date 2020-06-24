---
layout: post
title: "模型融合"
date: 2016-06-25 11:35:29 +0800
comments: true
categories: [Machine Learning] 
keywords: 
description: 
---

这篇博客内容主要参考["Kaggle Ensemble Guide"](http://mlwave.com/kaggle-ensembling-guide/)，介绍了模型融合方面的一些相关技术（也“融合”我的一些理解）。很推荐直接阅读原文，相信对模型融合的概念会带来更多的了解。

Kaggle Ensembling Guide

对于机器学习相关的任务，模型融合是一种重要的提升学习准确率的方法。所谓融合，可以说是通过将不同模型的学习结果进行综合，使得最终的结果在准确率上超过单个模型的预测结果。

一. 为什么模型融合可以达到这种效果？

举一个简答的例子。假如我们要预测一个0-1序列，序列的真值是：

	1111111111
现在我们有3个预测准确率在70%的模型，分别的预测结果是：

	1110001111
	1010101111
	0111110011
我们简单的通过多数表决，即对每一位采用最多的结果作为最终结果，可以得到多数表决后的序列：

	1110101111
准确率达到80%对么

看到这里肯定会有一个疑问，这种结果应该也是凑巧吧，如果我的三个模型预测序列是一致的，那么准确率也不会提高。

是这样的，所以在我看来模型融合要求两个基本条件：

1.模型之间是不同的，每个模型会学习到不同的特征，这样融合才有意义，模型融合要求模型彼此从本质上的不同性。
2.模型准确率是相似的。如果把一个80%准确率的模型和一个20%准确率的模型融合，结果肯定还是会变差的，但如果是75%和80%的模型融合，结果很可能是会提高的。

小补充一点，实际应用当中的感觉是，差不多的模型进行容易，一般都会比单个模型有更好的结果。

二. 融合的层次

投票融合： 直接把不同模型的结果。具体的做法又可以分为几种：

1. 平均：模型1的结果A1， 模型2的结果A2， 最终结果= (A1+A2)/2。
2. 加权：按不同模型准确率的程度进行加权，越准确的模型有越高的权重，准确率可以通过交叉验证计算。

好吧，前面说了那么多模型融合牛逼，最后融合的方法就是这个？
当然不是，但我在这里要说两点：
1. 最简单的平均融合，虽然看起来比较low，但效果并不差，很多时候平均融合就可以取得很不错的效果了。
2. 当然有更高端的融合方式，请接着看下面。

还是先摘自“Kaggle Ensemble Guide”一段很有趣的解释：

	Averaging prediction files is nice and easy, but it’s not the only method that the top Kagglers are using. The serious gains start with stacking and blending. Hold on to your top-hats and petticoats: Here be dragons. With 7 heads. Standing on top of 30 other dragons.

哈哈，再举一个例子，Netflix竞赛算是机器学习比赛的鼻祖了，其中winner的方法就是blending。（ps，还有一个槽点是，Netflix并没有最终使用winner的方法，因为。。在实际中无法使用。pss，这也是Kaggle被吐槽很重要的一个原因，它离实际问题的解决太远了）。

Stack Generalization：首先用一系列分类器对原始数据做分类，然后再用一个分类器去聚合这些分类器分类的结果，总体的降低误差。

Blending：和Stack Generalization很相似，相比Stack，它的好处和坏处可以看[原文](http://mlwave.com/kaggle-ensembling-guide/)。

具体是怎么做的呢，举个例子，代码在[这里](https://github.com/emanuele/kaggle_pbr/blob/master/blend.py)：

	np.random.seed(0) # seed to shuffle the train set

    n_folds = 10
    verbose = True
    shuffle = False

    X, y, X_submission = load_data.load()

    if shuffle:
        idx = np.random.permutation(y.size)
        X = X[idx]
        y = y[idx]

    skf = list(StratifiedKFold(y, n_folds))

    clfs = [RandomForestClassifier(n_estimators=100, n_jobs=-1, criterion='gini'),
            RandomForestClassifier(n_estimators=100, n_jobs=-1, criterion='entropy'),
            ExtraTreesClassifier(n_estimators=100, n_jobs=-1, criterion='gini'),
            ExtraTreesClassifier(n_estimators=100, n_jobs=-1, criterion='entropy'),
            GradientBoostingClassifier(learn_rate=0.05, subsample=0.5, max_depth=6, n_estimators=50)]

    print "Creating train and test sets for blending."
    
    dataset_blend_train = np.zeros((X.shape[0], len(clfs)))
    dataset_blend_test = np.zeros((X_submission.shape[0], len(clfs)))
    
    for j, clf in enumerate(clfs):
        print j, clf
        dataset_blend_test_j = np.zeros((X_submission.shape[0], len(skf)))
        for i, (train, test) in enumerate(skf):
            print "Fold", i
            X_train = X[train]
            y_train = y[train]
            X_test = X[test]
            y_test = y[test]
            clf.fit(X_train, y_train)
            y_submission = clf.predict_proba(X_test)[:,1]
            dataset_blend_train[test, j] = y_submission
            dataset_blend_test_j[:, i] = clf.predict_proba(X_submission)[:,1]
        dataset_blend_test[:,j] = dataset_blend_test_j.mean(1)

    print
    print "Blending."
    clf = LogisticRegression()
    clf.fit(dataset_blend_train, y)
    y_submission = clf.predict_proba(dataset_blend_test)[:,1]

    print "Linear stretch of predictions to [0,1]"
    y_submission = (y_submission - y_submission.min()) / (y_submission.max() - y_submission.min())

操作步骤：

1.把训练集的数据分组 （即上面的把训练集分为X\_train和X\_Test）
2.遍历所有model （即上面的clfs遍历），训练model
3.对每个model，预测X\_train的结果，把结果存储作为第一层的输出
4.对于测试集的数据采用一样的model预测
5.把第3步的输出作为输入，和最终的输出一起训练第二层的模型
6.用第二层的模型最终预测测试集的结果

博客原文还有很多内容，但我没有看得更多，因此先写到这里，感兴趣的话还是推荐阅读[原文](http://mlwave.com/kaggle-ensembling-guide/)