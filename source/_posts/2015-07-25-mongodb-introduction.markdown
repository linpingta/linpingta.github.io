---
layout: post
title: "mongodb-introduction"
date: 2016-05-25 21:15:57 +0800
comments: true
categories: [MongoDB] 
keywords: 
description: 
---

工作中用到的NoSQL主要是Redis，但MongoDB作为一种很重要的NoSQL数据库，对它一直也有好奇，这里简单（实际是非常简单）的梳理下MongoDB的相关知识，如果以后用到再做深入了解。

一. MongoDB vs Redis
1. MongoDB是基于文档的存储，Redis是基于键值的存储
2. MongoDB更接近传统SQL，Redis更接近Memcached
3. MongoDB更适合持久化存储，Redis适合做缓存
4. MongoDB更适合复杂的查询，Redis适合简单的查询

二. MongoDB定义
基于文档的存储，比较适于文档类型的信息存储，比如一篇包含评论的博客，可以把博客视为一种文档，评论视为其中的子文档保存在文档中。

MongoDB中的一条记录就是一个文档，MongoDB的文档很类似JSON对象，文档中value的内容可能是另外一个文档，组合或者文档的组合。

一条文档记录的格式如下，很类似一个JSON对象：

	{
		name:"linpingta",
		age:27,
		work:'Y',
		groups: ['BJ', 'China']
	}

三. MongoDB安装与启动

MongoDB的官网看起来有点乱，似乎有很多商业应用方面的产品，下载可以参考[这里](https://www.mongodb.com/download-center#community)。

安装完成后，在命令行运行mongod，默认将在本机的20717端口对外提供服务。

四.客户端支持

MongoDB官方支持主要语言的客户端，包括C++/Java/Python/NodeJS/C#。

五.特性

1.high performance：高性能的持久化存储
2.replica set：高可用，支持复制，数据冗余
3.horizontal scalability：支持数据的水平扩展
4.rich query language：CRUD操作

六.数据结构

MongoDB通过BSON格式存储文档记录，保存在collection中。一个database中可能包含多个collection，每个collection则包含了具体的document。
	
