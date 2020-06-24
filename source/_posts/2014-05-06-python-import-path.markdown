---
layout: post
title: "Python Import 路径"
date: 2014-05-06 23:25:26 +0800
comments: true
categories: [Python]
---

基本的import path可见 http://mobile.51cto.com/symbian-272844.htm

    (1) import 同模块下另一文件
          import file1
          
    (2) import 子模块中文件
          import mod1.file1
    
    (3) 子模块mod2中 import 同级别 mod1中文件
          这里需要将mod1和mod2的父路径加入sys.path，以便可以搜索

但我在开发中遇到如下的文件结构：

    dir
    --mod1
        --sub_mod1
            --sub_sub_mod1
                --sub_sub_sub_mod1
                    sub3_mod1.py
                --sub_sub_sub_mod2
                    sub3_mod2.py
    --mod2
        mod2.py
    
现在我需要在mod2.py中import sub3_mod2.py中的类LvSubMod2, 而 sub3_mod2.py中import sub3_mod1.py中的类LvSubMod1.

因为mod2.py和sub_sub_mod1中的目录不在同一级别，采用基本方法3可以处理sub3_mod1和sub3_mod2的关系，但是在Django调用时会提示下面的错误:

![TEST1](/images/2014/05/python_path_1.jpg)

其中/xc/input是mod2.py对应的目录，Django没有提示找不到mod2.py直接引用的sub3_mod2，而是提示找不到sub3_mod1. 我开始认为是sub3_mod1和sub3_mod2之间的引用有问题，但是通过单独建立如上的目录结构，发现问题出现在mod2的sys.path设置上

mod2的sys.path不能简单设置为当前mod2.py所在目录的父目录，因为在父目录下找不到sub3_mod1和sub_mod2，而是需要直接设置到sub3_mod1和sub_mod2的父目录上，即sub_sub_mod1

在mod2.py中

    import sys,os
    sys.path.append(os.path.abspath("../mod1/sub_mod1/sub_sub_mod1"))

问题解决。