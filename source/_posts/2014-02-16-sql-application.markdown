---
layout: post
title: "SQL应用"
date: 2014-02-16 19:53:32 +0800
comments: true
categories: [StackOverflow]
---

参考
http://stackoverflow.com/questions/21770257/querying-sql-server-2005

需求：
根据original table : timemilemaker
    08:00 101.2
	08:45 109.8
	09:15 109.8
	09:30 111.0
	10:00 114.6
生成new table 如下
	08:00-08:45 101.1-109.8
	08:45-09:15 109.8-109.8
	09:15-09:30 109.8-111.0
	09:30-10:00 111.0-114.6
所以重点是要根据现有列插入新列，方法是用select子句，通常select子句出现在from或where里面作为derived table，但这种情况下可以出现在select里面去选择特殊行

    select t1.*, (select t1.time from timemilemaker t2 where t2.time > t1.time order by t1.time) as newtime ...


