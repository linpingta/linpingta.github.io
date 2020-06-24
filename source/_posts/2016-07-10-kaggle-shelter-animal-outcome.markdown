---
layout: post
title: "kaggle-shelter-animal-outcome"
date: 2016-07-10 15:31:14 +0800
comments: true
categories: [Kaggle, Machine Learning, Pandas, Github Project] 
keywords: 
description: 
---

这是一篇关于[Kaggle:Shelter Animal Outcomes](https://www.kaggle.com/c/shelter-animal-outcomes)比赛的总结。

###比赛[背景](https://www.kaggle.com/c/shelter-animal-outcomes)
比赛是基于美国一个州过去三年宠物收容所对宠物的记录数据，去预测宠物的最终命运。它本身是一个预测问题，用到的自变量包括官方提供的以下特征：

	AnimalID,Name,DateTime,OutcomeType,OutcomeSubtype,AnimalType,SexuponOutcome,AgeuponOutcome,Breed,Color

去预测宠物的最终命运，可能的选项包括：

	Adoption，Died，Euthanasia，Return_to_owner, Transfer

用到的最终评价标准是[multiclass logloss](https://www.kaggle.com/c/shelter-animal-outcomes/details/evaluation)，简单来说就是它需要输出每个测试样本属于每类的可能性，而不是最有可能的类别，同时会更大的惩罚那些肯定性判断 （把每个记录以100%的输出错分会受到很大的惩罚），实际上最终leaderboard中那些误差大于3的提交结果应该都是输出了错误的判决结果。

###比赛结果

截止本文完成时，我的排名是 54th/1289，比赛还有20天结束，但我已不打算进一步优化结果，最终结果应该会在结束后有所变化，我会在相应时间更新。

###最终方案

####特征选择和处理

我最终使用的特征包括：

* 时间特征：DateTime部分被处理为5个部分
	*  YEAR/MONTH/WEEKDAY/HOUR/UNIXTIME
* 年龄特征：原始的年龄特征包含如“3 months”, "2 weeks"等，全部转换为天为单位的表示，原始数据中部分动物没有年龄，相应数据被转为0（也看到过一些根据其它信息去预测年龄做填充的方案，没有尝试）
* 名称特征：原始名称转为是否有名称，名称的长度两个新特征
* 性别特征：原始性别实际包含Male/Female以及是否被阉割Intact的信息，转为Male/Female和Intact/Not Intact两个新特征
* 品种特征：首先把训练集和测试集的所有品种做统计，因为记录中每个动物都可能属于多个品种，因此把属于某品种记为1，不属于记为0
* 颜色特征：处理方法类似品种，和品种的区别是，品种会把是否包含Mix和/作为分隔符，颜色会把空格作为分隔符
* 去除处理过的原始特征，以及Animal ID信息，对测试集编码后（scikit-learn fit_transform），采用相同的编码方式处理训练集

####模型选择

最终我使用的是由ChenTianqi（中国人，非常厉害）开发的[XGBoost](https://github.com/dmlc/xgboost)作为学习模型，如XGBoost名称所示，它本身就是一种boosting方法，在我的测试中，它的效果要比RandomForest更好。

最终的参数：

	param = {'max_depth':11, 'eta':0.03, 'subsample':0.75, 'colsample_bytree':0.85, 'eval_metric':'mlogloss', 'objective':'multi:softprob', 'num_class':5, 'verbose':1}
    num_round = 400
	dtrain = xgb.DMatrix(train_x, label=train_y)
    clf = xgb.train(param, dtrain, num_round)

关于XGBoost的介绍，首先参考它的[Github](https://github.com/dmlc/xgboost/tree/master/python-package)（包含使用的Example），其次关于参数调优可以参考[这篇文章](http://www.analyticsvidhya.com/blog/2016/03/complete-guide-parameter-tuning-xgboost-with-codes-python/)

PS. XGBoost本身是基于C开发的，它提供了R和Python两种接口，就我看到的情况看，R语言的接口相比Python会更方便一些，如果将来有机会，我还会对XGBoost做更多的探索。

###我做过的尝试

我相信最终的方案并不是这次比赛的全部收获（或者更直白的说，只是很少一部分），作为第一个真正认真参加的Kaggle比赛，我想记录一个Kaggle新手在处理问题的过程，对阅读博客的人有更多帮助。

下面的尝试我大致按时间记录：
1. 第一次：几乎没有处理原始数据，只是去除无用的字段（比如OutcomeSubtype, AnimalID）用RandomForest直接训练，得到的logloss是13.8：这里的问题在于，没有看清题目的损失评价方法是logloss，直接用的分类树而不是回归树，给出一堆0，1结果
2. 第二次：仅仅是更新损失评价方法和树类别，logloss下降到3.4
3. 第三次：开始真正意义上的数据清理，包括如下尝试：

	* 处理Age信息，把年月日数据统一为天表示
	* 尝试把猫和狗作为独立的对象分别训练
	* 处理Datetime，转为年月，weekday以及几点钟
	* 添加本地的cross validation
	这个阶段结束后，logloss下降到1.8
4.第四次：回归模型参数，发现在RandomForest中使用的树数量和树深度都太少太浅，调整参数，logloss下降到0.98
5.第五次：增加名称信息，增加Sex分隔处理，处理品种和颜色是否混合的信息
6.第六次：开始使用Grid Search做参数空间的搜索，经过参数优化后，logloss大概在0.82左右
7.第七次：用XGBoost替换RandomForest，调整相关参数，之后又对年龄特征做了细化，logloss达到0.75左右
8.这之后其实有一段很长的时间，logloss并没有什么提高，这期间我尝试过用KNN（没有太多调优，误差在0.98），尝试过[模型融合](http://linpingta.cn/blog/2016/06/25/model-stacking/#disqus_thread)，还尝试过使用场外特征，但是都没有在logloss上面有明显的改善（场外还是有一些改善，但并不合法还是放弃了）
9.最后的改进：在特征中加入UnixDateTime特征，以及把XGBoost算法从scikit-learn版本更改为它的原生版本，logloss达到0.72，再更新参数和训练周期(num_round)，最佳logloss达到0.708。

###项目代码

[Github地址](https://github.com/linpingta/shelter-animal-outcome)

###经验和教训

大致来说，logloss的改进经过“特征处理->模型调参->特征处理”这样不断循环的过程，在模型大致可行的前提下，特征工程，指的是发现新特征，或者处理原有特征，会对结果又明显的改善，但是这句话的前提是模型大致可行，也就是说模型本身的参数，以及选择哪个模型，仍然是非常重要的。
结论性的来说，模型和特征是解决问题的两个重要因素，哪个弄不动了就试试另外一个，通常会不断提升结果。
这次也对模型融合做了一些尝试，效果不是很好，但相信以后会有更好的结果。

###关于Kaggle

其实关于kaggle有没有用，网上有很多说法，但主要的说法是，你要指望通过kaggle去做data science，是不太靠谱的。因为它的数据集本身是比较干净的（虽然也有缺省值要处理），问题的目标也是比较清晰的（损失函数都告诉你了嘛），而实际工作中，包括我自己遇到的问题里，如何去定义问题，去处理数据，往往是问题中最困难的（其实定义问题是最困难的，很多问题并没有那么明显的损失函数去描述）。
我觉得这种说法是很有道理的，所以我想说kaggle的比赛结果并不能说明什么，但为什么我还愿意参加这样的比赛，我想是基于以下的理由：

	1. 接触那些与工作内容完全不同的问题：Kaggle上有各类机器学习的问题，分类回归聚类，图像NLP，一个人的实际工作中其实很难接触到太多方面，这里是个好途径
    2. 学习新的模型，新的方法：恰恰是因为不用去做数据提取和预处理的工作，可以让人更专注于模型本身。我可以学习到新的模型，新的调参方法，这些或许在将来会给予工作帮助
    3. 它本身有一定的意义。Kaggle比赛不能排除作弊，但它毕竟是一个很明确的问题。从我不长时间的面试和被面试经验看，机器学习相关的项目经验，由于业务本身特点（内部工具或者多人合作），其实是很难在面试很短时间内描述清楚的。Kaggle的比赛至少可以说明一个人在运用模型方面的基本能力是get了，因为Kaggle的比赛是一定包括训练集测试集特征处理交叉验证这些基础的方法的。我不认为Kaggle的优胜者一定是一个好的data engineer，但至少说明他在数据这个大领域的某些方面还不错。

最后说一点，除了以上的经验，我根据一般模型训练的步骤开发了一套[框架](https://github.com/linpingta/tools/tree/master/model_trainer)，可以方便以后使用。