---
layout: post
title: "redis.conf"
date: 2014-06-28 19:41:16 +0800
comments: true
categories: [Redis]
---

redis.conf 关于redis的服务配置，类似Apache的http.conf

     其中例如
     1. 关于snapshotting的设置
          save time num
     for example, save 900 1 表示如果900s内有一个key发生改变，那么就写到硬盘
     2. work directory的设置     
     dir ./ 默认是当前目录
     3. replication
     master-slave replication 其中slave可以有自己单独的备份设置 （比如间隔多久写到硬盘）
     slaveof <masterip> <masterport>
     4. limits
     maxclients 
     最大客户连接数，默认是0，即无上限，那么连接数取决于redis能够获得的file descriptor的上限
     maxmemory
     可以使用的最大内存
     max memory policy
     当max memory达到时，进行替换的原则 （可以理解为cache替换的原则）
     5. append only mode (5中的操作是对数据安全要求较高的操作，默认no，即采用1的策略)
     默认mode是异步写入磁盘，那么可能存在crash导致最近数据丢失的情况
     采用append only mode，以速度损失为代价，保证数据不丢失，临时文件会存储到appendonly.aof中
          
     append策略
          1. always 速度最慢但最安全，每次写操作都写入append.aof
          2. everysec 在速度和安全性间平衡
          3. no 由操作系统决定何时写入（buffer满），速度最快
     6. slow log 
     慢日志 (对应MySQL中的slow_query_log)
          可以做的设置包括
               被记录命令的时间间隔标准 (因为慢日志只会记录花费超过一定时间的查询)
               最大的日志size
     7. virutal memory
     在2.4以上的版本，VM已经被废止
     virtual memory的存在主要是满足数据使用的2/8定律，即20%的数据被访问的次数占到80%，但是另外80%的数据仍然需要存储，对于内存不足的情况，那么使用硬盘作为VM，另外，作者推荐用SSD作为硬盘（因为其随机访问时间短）

