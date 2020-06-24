---
layout: post
title: "Python: Pass by assignment"
date: 2013-12-26 16:56:19 +0800
comments: true
categories: [Python]
---

### *问题缘由*
函数传参是一项非常基础的问题，在C++中，我们有两种传参方式，分别是传值(pass-by-value)和传引用(pass-by-reference)，二者可以用引用(&)或者指针(*)区分。

    void F(int a); // pass by value
    void F(int* a); // pass by reference
    void F(int& a); // pass by reference

在Python中，没有引用和指针符号，那么Python函数的参数在函数内部的修改是否会在外部生效呢？

### *Python的做法: Pass by assignment*

实际上Python采取的做法既不是pass by value也不是pass by reference，我们称呼它为[pass by assignment](http://stackoverflow.com/questions/986006/python-how-do-i-pass-a-variable-by-reference)
简而言之，函数内是否可以修改变量的值取决于变量本身。
如果变量是immutable类型，比如string，那么函数内的修改不能影响外部，即pass-by-value，而如果变量是mutable类型，比如list，那么函数内部的修改可能会影响到外部，前提是引用修改的是原对象的值，即pass-by-reference.

这里举两个具体的例子说明

string

    class Foo:
        @staticmethod
        def f(word):
            word = 'Hello, Var changes'
        
    s1 = 'Hello, Python'
    print s1
    Foo.f()
    print s1

输出结果：

![TEST](/images/2013/12/python_pass_by_assign_1.png)
	
list

    class Foo:
    @staticmethod
    def f2(my_list):
        my_list.append('new')

    if __name__ == '__main__':
        list1 = ['old']
        print [item1 for item1 in list1]
        Foo.f2(list1)
        print [item1 for item1 in list1]

输出结果:

![TEST2](/images/2013/12/python_pass_by_assign_2.png)
	
那么接下来解释下“前提是引用修改的是原对象的值”。Python的所有对象都是引用，引用就会存在一个引用的对象，好比你可以让pointer* A指向objectA，也可以指向objectB (借用下C++的概念)，那么在函数内部，如果引用的对象没有改变，而改变的是引用对象的值，那么这种修改会被反映到外部，反之，如果引用的对象改变了，那么进一步的值改变将不是在原先引用对象上进行的，那么这种改变不会反映到外部。
举一个具体的例子：

    class Foo:
    @staticmethod
    def f2(my_list):
        #my_list.append('new')
        # change reference
        my_ist = ['A','B']

此时my_list的新赋值不会影响函数外部的取值，结果如下：

![TEST3](/images/2013/12/python_pass_by_assign_3.png)