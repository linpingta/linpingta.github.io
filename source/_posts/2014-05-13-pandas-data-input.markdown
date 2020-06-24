---
layout: post
title: "Pandas 数据准备篇"
date: 2014-05-13 23:20:55 +0800
comments: true
categories: [Python,Pandas]
---

从csv或者文本文件读取数据
     
     import pandas as pd
     
     r = pd.read_csv('example.csv') # 读取csv文件，默认第一行是header column
     r = pd.read_csv('example.csv', header=None) # 读取csv，第一行也是数据
     r = pd.read_table('example.csv', seperator=',') # 读取文件，按逗号分隔

     pd.isnull(r) # return True 对于NaN，-1.#IND和NULL的位置，其它return False
     因此可以用来区分数据有效性
     也可以通过调用时指定na_values来定义对用户的无效数据
     
     pd.read_csv('example.csv', na_values=['NULL'])
     
     na_values 可以进一步扩展为每列不同
     na_values = {'column1':['NA'],'column2':['NULL']}

     #读取部分文件 (前N行数据)
     pd.read_csv('example.csv',nrows=N)
     
     #将DataFrame中的数据输出到csv
     DataFrame.to_csv('output.csv')

     pd.date_range(start,end,period) # pandas data_range 指定了起始日期，结束日期或者持续长度
     
从数据库里导入数据

     import MySQLdb
     import pandas.io.sql as sql

     #conn 用于建立数据连接
     sql.read_frame('select * from your_table', conn)

例如:
     
     conn = MySQLdb.connect(host='localhost',user='root',passwd='pass',db='tablename',port=3306,charset='utf8')
     r = sql.read_frame('select * from dw_year', conn)

     >>>
     YEAR_ID  YEAR  PRE_YEAR_ID
    0        1  2010          NaN
    1        2  2011            1
    2        3  2012            2
    3        4  2013            3
    4        5  2014            4
    5        6  2015            5

数据连接
pandas.merge: 将DateFrames按照一定的key进行连接

pandas将dict转为DataFrame: key作为column，value作为每行元素（通常是list）
          
     df1 = DataFrame({'key1': range(2), 'key2':range(2)})
     
pandas.merge(df1,df2) 默认按照df1和df2中公有名称的key进行merge
     或者显式调用

     pandas.merge(df1,df2,on='keyname')
    
     
如果在df1和df2中key名称并不相同，那么可以指定merge的left key和right key
     
     pandas.merge(df1,df2,left_on='lkey',right_on='rkey')

可以通过how来指定join是left join, right join还是outer join

     pd.merge(df1, df2, how='outer')
     
看起来merge并没有复杂的操作，实际上是把sql的join操作在DataFrame level上重新进行了一次实现
     
merge on index : 如果在定义DataFrame的时候用到了index （比如DataFrame({}, index=['a','b'])）那么可以通过left_index=True或者right_index=True来指定使用index而非后面的column进行匹配（实际上这里的index是一个特殊的column）