---
layout: post
title: "广告成本（CPA）预测模型初步"
date: 2016-10-23 09:57:40 +0800
comments: true
categories: [Machine Learning, Spark] 
keywords: 
description: 
---


##广告成本（CPA）预测模型初步

###实验背景
通过投放管理，我们收集了相当数量广告主的广告投放数据。Facebook广告在设定时包含相当数量的定向信息，最常用的比如国家，性别和年龄。广告投放往往在相同定向上有相似的表现效果（比如同一款游戏投放美国和东南亚，前者的CPA一定比后者高），因此我们可以尝试用历史数据预测广告CPA的未来表现。

###实验目标
希望依据广告投放的历史数据，构建回归模型，预测广告当前的CPA表现，指导模型投放。

###分析工具
考虑到数据源直接存储在Hive中，同时也考虑数据量可能的增长预测，我们采用Spark做数据处理和模型预测方面的工作。
Spark本身是基于RDD实现的内存计算框架，它的一个重要应用在于处理机器学习中迭代问题的优化，相对应的库为Spark ml。多说一句的是 ，Spark版本更新较快，相应机器学习库存在ml和mllib两个版本，分别基于DataFrame和RDD处理数据。实际上，DataFame可以视为对RDD的封装，在1.3版本前，它也被称为schema RDD，即包含元数据描述的RDD。
我们选择ml和DataFrame操作数据，因为它们在后续更新的优先级会高于mllib。

###数据源介绍
我们把离线模块拥有的数据源信息划分为广告维度，产品维度，时间维度，广告特征维度，定向特征维度，创意维度和统计维度这几个部分。
技术上，它们来自Hive表offlin_data，表内容每天会定时加载更新。

广告维度字段 (字段名称我只补充了部分需要说明的，因为大多数字段都是自表意的)
| id | description |
|---|:---|:---:|---:|
|Bm_id	|Business Manager|
|Account_id|	 
|Campaign_id|	 
|Adset_id|	 
|Ad_id|	 
 
产品维度

| id | description |
|---|:---|:---:|---:|
|Application_id / promotion_url|	App链接|
|Product_type|	App类型，提取自user_os|
|Game_type|	应用类型，需单独处理|
 
时间维度
| id | description |
|---|:---|:---:|---:|
|Dt / start_dt / end_dt|	数据分析的最小单位是天|
 
广告特征维度

| id | description |
|---|:---|:---:|---:|
|Bid_amount|	广告的出价
|Daily_budget|	广告的日预算，如果设置
|Lifetime_budget|	广告的生命周期预算，如果设置
|Is_autobid|	是否是autobid
|Relevance_score|	广告的质量得分|
|Page_id|	广告使用的page信息
 
定向特征维度
| id | description |
|---|:---|:---:|---:|
|Min_age / max_age	 
|gender	 
|country	 
|city	我们抓取了city信息，如果有的话
|Custom_audience	 
|Connection	 
|Exclude_connection	 
|Interests	 
|…	 
完整的定向特征保持与PE上adset可编辑的定向内容完全一致，这里不一一列出
 
创意维度
| id | description |
|---|:---|:---:|---:|
|Video_id / video_image_url	|Video的FB ID
|Image_url / image_hash	|Image的链接
 
统计维度
| id | description |
|---|:---|:---:|---:|
|Impression / reach	 
|Clicks	 
|Actions	 
|Spend	 
|Unique_click/unique_impression	 
 
创意分为视频和图片两类，出价方式分为使用autobid和不使用autobid，本次分析只针对图片类创意，不使用autobid的游戏类广告。

Spark中对数据加载通过Hive QL，如下语句：

	conf = SparkConf().setAppName("offline-train")
	sc = SparkContext(conf=conf)
	hc = HiveContext(sc)

	# generate your query_sql
	df = hc.sql(query_sql)

###数据预处理
1.考虑在数据量极小时，CPA的绝对值没有意义（比如一共只有几十美分的花费就带来安装并不能说明说明当前定向下真实成本就是几十美分），因此我们过滤掉单日花费小于20$的广告，因为我们认为在太少的花费下，CPA是不可信的。
2.对autobid的广告，数据源中会把对应的bid_amount设置为0，同时，广告在应用和游戏上通常也有着不同的表现，目前我们没有方法直接判断一个promotion_url是游戏还是应用（后续拟添加支持），但通常应用的出价范围和游戏不一致，因此我们选择bid_amount>50美分的ad作为样本对象。
3.回归中较常用的误差衡量方式是RMSE，一些异常点可能对RMSE的结果影响较大，根据经验，因此我们选择天级别实际cpa小于20的ad作为样本，将cpa大于20的ad视为异常数据。
4.因为我们只分析图片创意，因此我们选择image_url非空的case
5.因为后续OneHotEncoder不接受空字符串，因此把空字符串均转为”unknown”

经过以上过滤后，7天内有效数据量占总记录条数的比例约为2%。

###特征工程
首先，我们经数据预处理后，从数据源中选择了如下原始特征：

dimensions部分

		Image_url
		Dt
		Age_min
		Age_max
		Is_autobid
		Genders
		Product_type
		Countries
		Bid_amount
		Daily_budget
		Connection_ids
		Custom_audience_ids
		Excluded_connection_ids
		Interest_ids
		Billing_event
metric部分

		Spend
		Mobile_app_install

metric部分只用于计算cpa，通过Hive定义的udf完成。

	UdfFunc = udf(udf_func, FloatType())
	df = df.withColumn(“cpa”, UdfFunc(df[“spend”], df[“mobile_app_install”]))

1.把age_min和age_max合并为age_avg，然后在最终输入模型的训练特征中去除age_min和age_max：

	age_avg = (age_min + age_max) / 2
2.把dt信息转为weekday，在最终模型中去除dt信息：

	def weekday(dt):
	    if not dt:
	        return '-1'
	    import datetime
	    d = datetime.datetime.strptime(str(dt), '%Y%m%d')
		return int(d.weekday())
3.对于如countries, connection_ids，可能在一条记录中包含多个国家或多个connection，我们只抽取其中第一条记录视为记录的代表：

	def countries_fill(countries):
	    if (not countries) or (countries == 'null'):
	        return 'unknown'
	    countries = countries.replace('::',',')
		return countries[0]

###特征编码
由于Spark.ml的树模型不接受字符串类型输入（与scikit-learn一致），因此我们对所有字符特征都作为string to index的转换，然后做了one hot encoder。

	# countries -> one-hot encoder, category
    countries_indexer = StringIndexer(inputCol="countries", outputCol="countries_index", handleInvalid="skip")
	countries_onehot_indexer = OneHotEncoder(dropLast=False, inputCol="countries_index", outputCol="countries_vec")

Spark.ml支持流水线（pipeline）的数据处理模式，需要我们把相关处理特征加入pipeline中。

	stages = [
	        countries_indexer, countries_onehot_indexer,
	]
	# finally
	pipeline = Pipeline(stages=stages)

最终将cpa视为label，将其它特征视为features：

	assembler = VectorAssembler(
        inputCols = feature_columns,
        outputCol = "features"
	)
	stages.append(assembler)

###模型选择
在Spark.ml的默认模型中，我们选择随机森林（Random Forest）作为回归预测模型。虽然从效果上讲，Random Forest可能不如其它ensemble方法，但它是强于decision tree的，考虑模型的可解释性，以及实现的便利性，我们直接选择RandomForestRegressor作为回归模型。

对于Random Forest，较为重要的两个参数是单棵树的深度(max depth of tree)和树的个数(num of trees)，对这两个参数通过grid-search确定合理值。

	# do cross-validation
    paramGrid = ParamGridBuilder()\
        .addGrid(random_forest.maxDepth, [3, 5, 7, 9])\
        .addGrid(random_forest.numTrees, [100])\
        .build()
    evaluator = RegressionEvaluator()

    cv = CrossValidator(estimator=pipeline,
        estimatorParamMaps=paramGrid,
        evaluator=evaluator,
        numFolds=2)
    model = cv.fit(df)
    print model.avgMetrics

最终模型选择14天数据为训练数据，1天数据为预测数据，RMSE大约在1.59
预测cpa和实际cpa输出样例（前50条）：

|        prediction|    label|            features|
|---|:---|:---:|---:|
|1.5217560913620043|1.4385713|(1042,[0,1,2,3,17...|
|2.3752164632800854|5.8022223|(1042,[0,1,2,3,19...|
| 2.411657767271753|3.0307143|(1042,[0,1,2,3,42...|
|3.6899882643509763|3.5722387|(1042,[0,1,2,3,50...|
| 4.092828817916082|2.2818182|(1042,[0,1,2,3,55...|
| 4.197697715305045| 3.366192|(1042,[0,1,2,3,52...|
| 4.197697715305045|4.0157895|(1042,[0,1,2,3,46...|
| 4.197697715305045|3.4369998|(1042,[0,1,2,3,49...|
| 4.071716257839271| 1.917647|(1042,[0,1,2,3,44...|
|3.9807423344404818|2.6338665|(1042,[0,1,2,3,42...|
|3.9807423344404818|3.5842857|(1042,[0,1,2,3,42...|
| 4.177470410349176|1.9453847|(1042,[0,1,2,3,42...|
| 4.071716257839271|1.8128395|(1042,[0,1,2,3,44...|
|1.9035986906299776|1.7442857|(1042,[0,1,2,3,22...|
|13.808040417768252|   16.058|(1042,[0,1,2,3,27...|
|14.168610809794092|  14.8975|(1042,[0,1,2,3,18...|
|11.971184107912041|    10.89|(1042,[0,1,2,3,19...|
|11.962558073010637|     8.89|(1042,[0,1,2,3,30...|
|2.3169264529890903| 2.916579|(1042,[0,1,2,3,11...|
|2.3169264529890903|3.2772727|(1042,[0,1,2,3,11...|
|1.5221071103811983|6.3782606|(1042,[0,1,2,3,11...|
|1.3389387340836876|2.1707425|(1042,[0,1,2,3,11...|
|1.7074257356107376|1.8128985|(1042,[0,1,2,3,11...|
|1.8675894765662682|3.8840384|(1042,[0,1,2,3,11...|
| 2.241272660641193|4.9881816|(1042,[0,1,2,3,11...|
| 2.268769985708979| 2.851852|(1042,[0,1,2,3,11...|
|  2.16831561899445|3.3324242|(1042,[0,1,2,3,11...|
|  2.16831561899445| 4.152222|(1042,[0,1,2,3,11...|
| 2.292079711361142|3.9308271|(1042,[0,1,2,3,11...|
| 2.292079711361142|2.6325426|(1042,[0,1,2,3,11...|
|1.8987393163009587|2.9423833|(1042,[0,1,2,3,11...|
|1.8987393163009587|2.2502885|(1042,[0,1,2,3,11...|
|2.1668434345072054|4.4953847|(1042,[0,1,2,3,11...|
|2.1668434345072054|2.8725274|(1042,[0,1,2,3,11...|
| 2.268769985708979| 2.570139|(1042,[0,1,2,3,11...|
| 2.268769985708979| 3.188125|(1042,[0,1,2,3,11...|
|  2.16831561899445|1.9479818|(1042,[0,1,2,3,11...|
|  2.16831561899445|3.9414287|(1042,[0,1,2,3,11...|
|2.3165876740390625| 2.620024|(1042,[0,1,2,3,11...|
| 2.318769978387125|1.6355883|(1042,[0,1,2,3,11...|
| 2.146834324914294|1.5435859|(1042,[0,1,2,3,11...|
| 2.146834324914294|2.0905683|(1042,[0,1,2,3,11...|
| 2.361828624187834|2.4541378|(1042,[0,1,2,3,11...|
| 2.237440741142636|3.7431035|(1042,[0,1,2,3,11...|
| 2.237440741142636|1.1768421|(1042,[0,1,2,3,11...|
|2.2990649947244433|  2.50875|(1042,[0,1,2,3,11...|
|  2.35518238420236| 3.223077|(1042,[0,1,2,3,11...|
|1.9192676385516463|3.5867856|(1042,[0,1,2,3,11...|
|1.9192676385516463|2.2514102|(1042,[0,1,2,3,11...|
|2.3522288292714277|3.7342856|(1042,[0,1,2,3,11...|


###后续展望
就模型本身而言，在各个方面都还有很大的改进空间：

	处理数据集本身的局限性
	特征工程的方法
	模型的选择和参数调试



