---
layout: post
title: "factorization machine"
date: 2014-08-17 21:37:17 +0800
comments: true
categories: [Machine Learning]
---

FM:

factorization machine 是一种针对于 factorization model 的实现工具

factorization model要考虑输入变量$x_i$和$x_j$之间的关系，而且是以一种独立的形式表现的（pairwise interaction）
    $y(x) = w_0 + \sum_{j=1}^{p}w_jx_j + \sum_{j=1}^{p}\sum_{j'=j+1}^{p}x_jx_j'(\sum_{f=1}^{k}v_{j,f}v_{j',f})$
    
前面的部分和linear regression中一致，它的主要价值在于后面的部分包含了自变量之间的相互作用关系.在计算中可以转变为：
    $y(x) = w_0 + \sum_{j=1}^{p}w_jx_j + \frac{1}{2}\sum_{f=1}^{k}[(\sum_{j=1}^{p}v_{j,f}x_j)^{2} - \sum_{j=1}^{p}v_{j,f}^{2}x_j^{2}]$
    
目标函数：(损失函数)
    $OPT(s) = argmin_\theta\sum_{(x,y)\in S} (l(y(x|\theta),y)+\sum_{\theta \in \Theta} \lambda_\theta\theta^{2})$
    其中$l$表示损失函数，常用的损失函数定义包括least-square loss($l_2$)和logistic loss，前者用于regression("输出$y$是高斯分布"是最小二乘的等价基础)，后者用于classification。$\lambda$部分表示regularization，防止overfitting。将输入$x$分组，每组有自己的$\lambda$

常用的参数优化计算方法包括：SGD,ALS,MCMC

* SGD
stochastic gradient descent : [随机梯度下降](http://blog.csdn.net/lilyth_lilyth/article/details/8973972)
    对于指定参数的函数，求参数的最优解：
    $ w_{t+1} = w_t - η_t \bigtriangledown f(w_t) $
    这里的求导会对样本中所有点计算，如果样本很大，那么在每一步迭代中计算的消耗均很大（在实际问题中很可能是无法承受的），因此我们需要用一种近似的方法进行替代，有相近的结果同时能大大减少计算量，这就是随机梯度下降的由来。
    相比于批量梯度下降中：
    $ \bigtriangledown f(w_t) = \sum_{i=1}^{n}\bigtriangledown f(w_t,x_i)$
    采用随机梯度下降
    $ \bigtriangledown f(w_t) = g(w_t) $
    满足 $ E(g(w_t)) = \bigtriangledown f(w_t) $
    在每次迭代中用每个样本而非所有样本计算梯度下降的方向。

具体到此问题中，
    $\theta \leftarrow \theta - \eta(\frac{\delta}{\delta\theta}l(y(x),y) + 2\lambda_\theta\theta)$
    其中$\eta$表示learning rate

SGD方法的问题：

    Learning rate : SGD方法是否能够收敛很大程度上取决于learning rate，如果选择太大，那么不能收敛，如果太小，那么收敛太慢。
    Regularization : 表示对于overfit的控制。
    Initialization : 表示初始化条件下factorized interaction参数的确定, 这里作者用的是在N(0,omga)内的随机投点，因此omga也是需要确定的参数。

* ALS (alternating least squares)
相比于SGD，不需要$\eta$，只需要确定regularization的参数
但它只能用于regression，不能用于classfication (least square的前提假设是$y$符合Gaussian分布)

* MCMC
马尔科夫蒙托卡洛[方法](http://site.douban.com/182577/widget/notes/10567181/note/292072927/)
    蒙托卡洛方法：
    1. 将所求问题的解表示为概率统计模型的形式
    2. 用抽样得到概率统计

蒙特卡洛方法本身要求样本是独立的，而如果样本不独立，那么需要借助马尔科夫链进行抽样，MCMC就是为此出现的。另一方面，MCMC方法要求马氏链在t足够大的时候收敛到一个平稳分布。

reject sampling
    采样指的是通过一定量的取值，使得取值结果反映采样概率函数的分布。
    均匀采样是可以通过产生伪随机数实现的，那么任意分布$f(x)$，如何通过采样表达其分布呢？
    reject sampling方法可参考[wiki](http://en.wikipedia.org/wiki/Rejection_sampling)，[实现原则](http://www.stat.ucla.edu/~dinov/courses_students.dir/04/Spring/Stat233.dir/STAT233_notes.dir/RejectionSampling.pdf)
        证明采用这种策略采样得到的结果和希望得到的分布一致，重点在于$P(x\in A)$被转化为$P(x\in T|Accept)$，Accept这件事是通过$U<f(t)/M(t)$表示出来的

Markov Chain

FM是factorization model中的一种处理方法，与其它方法如[matrix factorization](https://datajobs.com/data-science-repo/Recommender-Systems-%5BNetflix%5D.pdf),pairwise interaction tensor factorization,[SVD++](http://blog.csdn.net/wanglulu2014/article/details/21170111)可以进行转换。

factorization model : 针对factorization相关的系列算法的统称，它们主要适用于推荐系统，包括SVD++,SVD等。
    SVD (Singular Value Decomposition)是将一个$N*M$的评分矩阵分解为一个$N*F$的用户因子矩阵和一个$M*F$的物品因子矩阵。同时考虑评分的偏离度（有些用户倾向于宽松的打分，有些倾向于严格的打分），最终u相对于i的打分$r_{u,i}$表示为
    $r_{u,i}=overallMean+b_u+b_i+p_uq_i^{T}$
    优化其结果：
    $argmin_\alpha\sum_{(u,i)\in \alpha}{(r_{u,i}-overallMean+b_u+b_i+p_uq_i^{T})^{2}+\lambda(\Arrowvert b_u^2\Arrowvert + \Arrowvert b_i^2\Arrowvert + \Arrowvert p_u^2\Arrowvert + \Arrowvert q_i^2\Arrowvert)}$

hyperparameter指的是初始化确定的经验参数，比如regularization中的$\lambda$.

activation function: 指的是通过output signal来反映input的activation level, 在neutral network中用到。

cross-entropy  error:
    cross-entropy定义: 对于给定的$p(x)$和$q(x)$，定义cross-entropy为：
    $H(p,q)=-\sum_x{p(x)log(q(x))}$
    对于分类问题，实际值$t_n$和理论计算值$y_n$的cross-entropy error是$-\sum{p_xlogq_x}$
    相比而言的另外一种error是classification error，具体参见[此篇](\Arrowvert b_u^2\Arrowvert)