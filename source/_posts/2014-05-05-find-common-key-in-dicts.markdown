---
layout: post
title: "find common key in dicts"
date: 2014-05-05 23:57:03 +0800
comments: true
categories: [Python]
---

找到两个dict的公共key (From Python Cookbook)
     
基本写法：
    
    for key1 in dict1.iterkeys():
        if dict2.has_key(key1):
            print key1
            
利用的基本函数： iterkeys() 遍历dict的key部分，dict1.has_key(key1)判断key1是否出现在dict1中
     
更优雅的写法：

    list1 = [for key1 in dict1 if key1 in dict2]
或者

    list1 = filter(dict1.has_key, dict2.keys())

下面两种写法：

     1.for key1 in dict1.keys():
            if key1 in dict2.keys():
                 print key1
    
     2.for key1 in dict1.keys():
            if dict2.has_key(key1):
                 print key1
在执行效率上有很大的区别
     
假设dict1含有N1个元素，dict2含有N2个元素
     
写法1会遍历dict1和dict2中的所有元素，因此执行效率是O(N1*N2)

写法2会遍历dict1，然后利用dict2中的hash table去查找元素，因此执行效率是O(N1)

另外据说filter(dict1.has_key, dict2.keys())会有比写法2更高一些的执行效率，但是原作者没有说明具体的原因，同时这种效率的差距极其微小（同一数量级），因此写法2是可以接受的