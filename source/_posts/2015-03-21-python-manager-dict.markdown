---
layout: post
title: "Python Manager dict"
date: 2015-03-21 09:56:44 +0800
comments: true
categories: [Python]
---

今天想介绍的是Python Manager dict，原因在于工作中遇到的一个问题，它的抽象化模式类型：

    from multiprocessing import Manager
    def f(d):
        d[1] = 1
        d[2] = {}
        d[2][2] = 3
    if __name__ == '__main__':
        d = Manager.dict() # 进程间共享dict
        p = Prcoess(target=f, args=(d,))
        p.start()
        p.join()
        print d

我们期望的输出是：
    
    #output d: {1:1, 2:{2:3}}

但实际的输出是：

    #output d: {1:1, 2:{}}

在实际问题中，因为涉及到进程间的数据共享，以及对于共享dict的添加和删除元素操作，然而，最终的输出并不符合设想，这里是什么问题呢？

首先，先简单介绍下Python中在进程间共享状态的方式：
    最常用的进程通信方式是Queue或者Pipe，共享状态并不是一种好的并行编程方式(when doing concurrent programming, it's usually best to avoid using shared state as far as possible)，但如果需要支持这种方式，Python提供了两种基础方法：
    (1) shared memory 共享内存
    (2) server process 服务进程

其中第一种方式例子如下：（支持Value和Array）

    from multiprocessing import Process, Value, Array
    def f(n, a):
        n.value = 3.1415927
        for i in range(len(a)):
            a[i] = -a[i]
    if __name__ == '__main__':
        num = Value('d', 0.0)
        arr = Array('i', range(10))
        p = Process(target=f, args=(num, arr))
        p.start()
        p.join()
        print num.value
        print arr[:]

第二种方式就是本文提到的Manager.dict，首先通过Manager对象用代理模式产生dict，它具有dict的特性，同时在进程间共享，相比shared memory, 它在数据结构上有更大的灵活性，但代价是速度会更慢。

这里，Manager对象用代理模式产生dict，这也是最开始问题的根源，代理模式是设计模式的一种，它为其它对象提供一种代理以控制对象的访问，比如需要控制真实对象的访问权限或者在真实对象调用前做一些处理。Manager.dict本身是一个proxied object。

关于这个问题的详细讨论可见[这里](https://bugs.python.org/issue6766)，我对这个问题的理解是，如果你是对Manager.dict做赋值操作，那么这种操作是在真实对象上发生的，而如果你对Manager.dict的值对象做操作（删除，添加），那么dict克隆了一个新对象，所有的操作均发生在新对象上，而原始对象不会被修改。

变通方法：
如上所述，赋值操作是发生在真实对象上的，因此可以先对希望的对象做操作，然后再赋值就好了。。回到最初的问题，解决方案：

    def f(d):
        d[1] = 1
        t = {}
        t[2] = 3
        d[2] = t

其实有点麻烦的，但也是暂时的处理方案了。。