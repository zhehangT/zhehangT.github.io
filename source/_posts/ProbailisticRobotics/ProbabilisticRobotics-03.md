---
title: 《Probabilistic Robotics》 读书笔记 03
date: 2016-10-09 09:35:00
tags: Probabilistic Robotics
categories:
- 机器人事业
- 读书笔记
description: 《Probabilistic Robotics》第三章节论述了对状态进行评估的重要方法：高斯滤波
---
<!-- more -->

# 摘要
用高斯滤波对状态进行递归评估是贝叶斯滤波最常用的实现方式。

多变量高斯分布的密度函数：
$p(x) = \mathbf {det}(2 \pi \Sigma)^{-\frac{1}{2}} \mathbf{exp} \\{ -\dfrac{1}{2}(x- \mu)^T \Sigma^{-1}(x-\mu)  \\}$
> $\Sigma 表示协方差， \mu表示均值$

接下来将从最基本的高斯滤波算法：卡尔曼滤波，扩展的卡尔曼滤波，信息滤波三个方面进行介绍。

# 卡尔曼滤波

卡尔曼滤波是学习贝叶斯滤波算法实现的最佳学习资料。卡尔曼滤波用于对线性系统的过滤和预测。只适用于连续状态的计算。利用卡尔曼滤波实现贝叶斯滤波必须满足以下三个假设：
1.状态转移概率函数 $p(x\_t \mid u\_t, x\_{t-1})$ 必须是线性函数,即：
$x\_t = A\_t x\_{t-1} + B\_tu\_t+ \epsilon _t$ （线性高斯模型）
2.测量概率函数 $p(z_t \mid x_t)$也是线性函数，即：
$z_t = C_tx_t + \delta _t$
3.初始可信度函数$bel(x_0)$满足高斯分布
$bel(x_0)=p(x_0)$

基于以上三个假设即可证明任意时刻的可信度函数$bel(x_t)$均满足高斯分布

卡尔曼滤波算法的流程如图所示：
![](1.png)
卡尔曼滤波的输出是t时刻的**状态可信度**$bel(x_t)$，通过均值$u_t$和协方差$\Sigma\_t$表示，输入是t-1时刻的**状态可信度**，通过均值$u\_{t-1}$和协方差$\Sigma\_{t-1}$表示，为了更新参数还需要t时刻的控制数据$\mu_t$和测量数据$z_t$
第二行和第三行根据**控制数据**计算当前时刻状态可信度的预测值。
第四行到第六行根据**测量数据**对预测值进行修正得到状态可信度。其中$K_t$称为卡尔曼增益。卡尔曼增益代表了观测数据对预测值的修正程度。$Q\_t$代表噪声$\delta _t$的方差。

# 扩展卡尔曼滤波
上述提到的卡尔曼滤波假设状态转移概率函数和测量概率函数都是线性的，而实际中这种假设往往是不成立的。因此提出扩展卡尔曼滤波来表示非线性情况。状态转移概率函数和测量概率函数表示如下：
$x\_t=g(u\_t,x\_{t-1})+\epsilon\_t$
$z_t=h(x_t)+\delta_t$

扩展卡尔曼滤波的关键就是对g和h函数进行线性化。常用的线性化方法有**泰勒展开**。
$g(u\_t,x\_{t-1})\approx g(u\_t,\mu\_{t-1})+g'(u\_t,\mu\_{t-1})(x\_{t-1}-\mu\_{t-1})\\\ \qquad \quad \ \   = g(u\_t,\mu\_{t-1}) + G\_t(x\_{t-1}-\mu\_{t-1}) $
> 其中$G_t$为雅可比行列式

$h(x\_t) \approx h(\bar \mu\_t)+h'(u\_t)(x\_t-\bar \mu\_t)
\\\ \quad \ \ \   = h(\bar \mu\_t) + H\_t(x\_t-\bar \mu\_t) $

扩展卡尔曼滤波算法的流程如图所示：
![](2.png)

扩展卡尔曼滤波的局限性在于通过泰勒展开来近似状态转移和观测，因此扩展卡尔曼滤波的准确程度依赖于状态转移和观测是否是线性近似的。
其他的线性近似手段包括**无损卡尔曼滤波**和**矩匹配**(moments matching)

# 信息滤波

信息滤波是高斯分布的对偶形式，与卡尔曼滤波和扩展卡尔曼滤波的区别在于：卡尔曼滤波和扩展卡尔曼滤波是通过均值和方差来代表高斯分布，而信息滤波是通过信息矩阵和信息向量来表示的。表示如下：
$\Omega = \Sigma^{-1}$
$\xi = \Sigma^{-1}\mu$

改写多变量高斯分布可得：
$p(x)=\mathbf{det}(2\pi\Sigma)^{-\frac{1}{2}} \mathbf{exp}\\{ -\frac{1}{2}\mu^T\Sigma^{-1}\mu \\} \mathbf{exp}\\{ -\frac{1}{2}x^T\Sigma^{-1}x+x^T\Sigma^{-1}\mu  \\} $

其中$\mathbf{det}(2\pi\Sigma)^{-\frac{1}{2}} \mathbf{exp}\\{ -\frac{1}{2}\mu^T\Sigma^{-1}\mu \\}$为与x无关的常数，因此用$\eta$表示，代入信息矩阵和信息向量可得：
$p(x)=\eta \ \mathbf{exp}\\{ -\frac{1}{2}x^T\Omega x + x^T \xi\\}$

信息滤波算法的流程如图所示：
![](3.png)

扩展信息滤波算法的流程如图所示：
![](4.png)


信息过滤有以下几个优缺点：
1.可以用$\Omega=0$表示全局的不确定性
2.适用于多机器人的问题。因为将其转化为负对数函数时，贝叶斯规则转化为加法形式
3.矩阵求逆，计算开销大

# 总结
本章介绍的卡尔曼滤波方法是实现贝叶斯算法最主要的算法之一，也是最成熟的算法之一。基于卡尔曼滤波针对特定的问题还有很多不同变种，希望在今后的学习中能加深理解。








