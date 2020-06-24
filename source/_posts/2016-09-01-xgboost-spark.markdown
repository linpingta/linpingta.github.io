---
layout: post
title: "Xgboost-Spark应用调研"
date: 2016-09-01 23:04:15 +0800
comments: true
categories: [Spark,Xgboost]
---

官方文档 https://spark.apache.org/docs/2.0.0/ml-guide.html
本文假设已正常安装spark和scala，其中spark为standalone模式。

1.官方的介绍资料在[这里](http://dmlc.ml/2016/03/14/xgboost4j-portable-distributed-xgboost-in-spark-flink-and-dataflow.html)

2.下载地址：https://github.com/dmlc/xgboost

3.样例运行
(1) 在完成上述下载后，进入git项目根目录，执行make编译源文件生成lib/xgboost.so
	在这一步中，如果遇到宏引用未定义错误，可能需要在对应文件中添加include或者宏定义。
(2) 进入jvm-package目录，执行mvn package编程生成jar包
	不同于spark官网的scala项目管理文件用sbt完成，xgboost项目采用maven完成包管理，如果没有安装maven，需先安装maven2
(3) 运行纯scala示例
    在上一步完成后，jvm-package/xgboost-example/target目录下已经生成了对应的jar文件，进入xgboost-example目录，以BoostFromPrediction为例，执行如下操作：

	scala -cp target/xgboost4j-example-0.7-jar-with-dependencies.jar ml.dmlc.xgboost4j.scala.example.BoostFromPrediction

在这里遇到的问题可能是”scala error: main method is not static“，原因是scala的main函数需要放在object对象下面而不是class对象下面来实现static，因此解决方法是把对应源文件的class对应替换为object并重新编译打包，正常的输出如下:
	
	[0]     train-error:0.046522    test-error:0.042831
	[1]     train-error:0.022263    test-error:0.021726
(4) 运行spark示例
完成上述编译后，在spark目录下提交任务：

	./bin/spark-submit --master local[4] --class ml.dmlc.xgboost4j.scala.example.spark.DistTrainWithSpark /home/test/xgboost/jvm-packages/xgboost4j-example/target/xgboost4j-example-0.7-jar-with-dependencies.jar 30 1 /home/test/xgboost/demo/data/agaricus.txt.train /home/test/xgboost/demo/data/agaricus.txt.test /home/test/xgboost/demo/data/result

xgboost目前只有一个spark示例，上述各项参数的含义在源码中有清晰描述.
