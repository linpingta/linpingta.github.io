---
layout: post
title: "SQL Optimization Example1"
date: 2014-02-16 20:18:12 +0800
comments: true
categories: [StackOverflow]
---

今天在stackoverflow上看到一个问题 http://stackoverflow.com/questions/21770055/can-i-simplify-the-sql ，优化：
     select count(*)  from (select *
          from T_LOGGINGINFO
          where to_char(LOGINTIME, 'YYYY-MM') = '2012-05'
          group by USERNAME)
问题本身并没有太大困难，但是这里涉及到SQL效率的优化，to_char(LOGINTIME,'YYYY-MM')='2012-05'是无法使用索引的，因为在上面应用了to_char函数，解决的方法是处理'2012-05'
          where LOGINTIME between to_date('01-MAY-2012 00:00:00', 'DD-MON-YYYY  HH24:MI:SS')
                                     and to_date('31-MAY-2012 23:59:59','DD-MON-YYYY HH23:MI:SS')

btw，作者的开场白很有意思：brevity is the soul of wit , but clarity is the soul of code. 果断Approve.
