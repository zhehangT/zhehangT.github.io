---
title: 《Probabilistic Robotics》 读书笔记 06
date: 2016-10-16 8:20:58
tags: Probabilistic Robotics
categories:
- 机器人事业
- 读书笔记
description: 《Probabilistic Robotics》第六章论述了对机器人的测量(measurement)进行建模
---
<!-- more -->

# 摘要
之前谈到在利用贝叶斯滤波计算状态置信度时会用到两个重要的概率：状态转移概率和测量概率。这一章节就是围绕如何计算测量概率展开的。并且主要基于距离传感器得到的测量数据，并根据机器人的位姿和地图信息进行测量概率的计算。


# 地图
地图信息是由环境中的物体和其位置组成的列表来表示的。用$m=\\{m\_1, m\_2,... ,m\_N \\}$表示。其中$N$表示环境中物体的总数。地图信息通常有两种索引方式：基于特征的表示方法和基于位置的表示方法。在基于特征的地图中，$n$是特征索引，每个$m$中包含特征和位置。特征地图提供了空间中所包含的所有物体的信息。在基于位置的地图中，$n$对应一个特定的位置。在平面地图中通常用$m\_{x,y}$表示。位置地图提供了整个空间中每个位置的信息。

占用栅格地图(occupancy grid map)是一种典型的位置地图。它为地图中的每个坐标位置赋予数值代表该位置是否已经被占用。

# 距离传感器的光线模型(Beam Models)

通过光线和声波可以测量机器人与周围物体之间的距离信息。常见的传感器有激光传感器和超声波传感器。这种测量模型会引入四种误差。
1.测量误差。受传感器本身精确度的影响，距离信息往往是有噪声的，这部分噪声对测量数据的影响用高斯模型$p\_{hit}$来表示。又因为距离传感器往往有最大测量范围，因此要将高斯模型的变量取值限定在这个范围之内，对范围之内的高斯概率做标准化处理。
$p\_{hit}(z\_t^k \mid x\_t, m)=\begin{cases}
\eta \ \mathcal N(z\_t^k;\ z\_t^{k\ast},\ \sigma\_{hit}^2) & if \ 0\leq z\_t^k \leq z\_max \\\
0 & otherwise
\end{cases}$
2.不可预测的物体。由于地图是固定的，而环境是动态变化的。当环境中出现新物体未被地图所包含，比如走动的人，则会造成误差。由于这部分误差会导致距离信息变短，因此用指数模型$p\_{short}$。还需要将指数模型的变量限定在比实际距离小的范围内，并对范围内的指数概率做标准化处理。
$p\_{short}(z\_t^k \mid x\_t, m)=\begin{cases}
\eta \ \lambda\_{short}e^{-\lambda\_{short} \ \ z\_t^k} & if \ 0\leq z\_t^k \leq z\_t^{k\ast} \\\
0 & otherwise
\end{cases}$
3.测量错误。由于障碍物表面材质问题或者测量角度问题，可能会导致测量失败，测量返回值往往是测量的最大值。这部分噪声用指标函数$p\_{max}$表示。
$p\_{max}(z\_t^k \mid x\_t, m)= I(z=z\_{max})=\begin{cases}
1 & if \ z=z\_{max} \\\
0 & otherwise
\end{cases}$
4.随机误差。最后为无法解释的随机误差。用$p\_{rand}$表示。
$p\_{rand}(z\_t^k \mid x\_t, m)=\begin{cases}
\frac{1}{z\_{max}} & if \ 0\leq z\_t^k < z\_{max} \\\
0 & otherwise
\end{cases}$

基于以上四种误差计算测量概率算法如下：
![](1.png)

其中输入为测量数据$z\_t$, 机器人的位姿$x\_t$ 和地图信息$m$，$z\_{hit},z\_{short},z\_{max},z\_{rand}$分别为各个误差的权重。输出为当前观测$z\_t$的概率。

在上述的模型中，存在很多未知的参数。
第一种参数设置的方式是根据经验手工设置参数。
第二种参数设置方式是根据数据得到参数。用$Z$表示一系列的测量数据，$X$表示一系列对应的位置数据，$\Theta$表示参数，因此只需要找到使$p(Z \mid X, m, \Theta)$概率最大的参数即可。具体算法如下图：
![](2.png)

关于算法的推导过程见书本6.3.3小节。


# 距离传感器的概率域模型(Likelihood Fields)

上述的光线模型存在的两个缺点：
1.通过光线模型计算出的测量概率在复杂场景下是不光滑的。即使机器人很小的位姿变化也会带来很大的测量概率的变化。这种不光滑会导致错失最优解或者陷入局部最优解。
2.计算开销大。

概率域模型的基本思想是计算出障碍物所处位置的全局坐标。要计算出这个全局坐标依赖于机器人的位姿，距离传感器在机器人上的具体位置以及距离传感器检测出与障碍物的距离。
与光线模型类似，概率域模型引入三种误差。
1.测量噪声。
$p\_{hit}(z\_t^k \mid x\_t, m)=\varepsilon\_{\sigma\_{hit}^2}(dist^2)$

2.测量错误。
3.随机误差。

概率域模型计算测量概率算法如下
![](3.png)

其中$(x\_{z\_t^k} \ y\_{z\_t^k})$为检测到的障碍物经过转换后在全局坐标系中的坐标。$(x' \ y')$是实际地图中最接近检测到的障碍物的实际障碍物的坐标。这个搜索过程也是该算法中开销最大的步骤。$z\_{random}$是随机误差的权重。

概率域模型的优点在于得到的概率分布是光滑的，并且预计算在二维空间中进行。但仍有几个不足。
1.没有考虑空间中的动态物体。
2.未考虑墙这种距离传感器无法穿过的物体。
3.无法在没有地图信息的环境中进行使用。
对于这些不足可以对概率域模型进行扩展。详见6.4.2小节。

# 基于关联的感知模型
基于关联的感知模型是通过地图匹配来实现的。首先将传感器数据转换成局部地图，用$m\_{local}$表示。再与全局地图$m$利用如下公式进行计算，得到一个关联度。

$\rho\_{m,\ m\_{local} \ \ ,  \ x\_t}=\dfrac{\sum\_{x,y}(m\_{x,y}-\bar m)\cdot(m\_{x,y,local}(x\_t)-\bar m)}{\sqrt{\sum\_{x,y}(m\_{x,y}-\bar m)^2(m\_{x,y,local}(x\_t)-\bar m)^2}}$

其中$\bar m$是地图的平均值
$\bar m = \dfrac{1}{2N} \sum\_{x,y}({m\_{x,y}+m\_{x,y,local}})$

如果局部地图是从测量数据中得到的，那么此时关联度即可作为测量概率。

概率域模型仅仅涵盖了末端的输出数据，并与空间中被占用的那部分空间关联。而地图匹配则涵盖了整个空间的信息，包括未被占用的那部分空间的空间信息。

# 基于特征的感知模型
上述模型都是基于原始的传感器测量数据的，基于特征的感知模型则先从这些原始数据中进行特征提取，这么做的好处是可以大大降低计算复杂性。对于不同的感知数据通常有不同的特征提取方式。

路标(Landmark)是机器人应用中常用的概念。通常通过提取感知数据的特征来对应环境中某些独特的物体，这些物体就被称为路标。通常用如下公式从测量数据中提取特征：
$f(z\_t)=\\{f\_t^1,f\_t^2,...\\}=\\{(r\_t^1, \phi\_t^1, s\_t^1), (r\_t^2, \phi\_t^2, s\_t^2),... \\}$
其中$r$代表距离，$\phi$代表方向，$s$表示签名(signature)。
通常认为特征之间是独立的。因此有：
$p(f(z\_t) \mid x\_t,m)=\prod\_i p(r\_t^i, \phi\_t^i, s\_t^i \mid x\_t, m)$

若知道测量数据与哪个路标相对应，则可用以下算法计算测量概率：
![](4.png)
其中$c\_t^i$表示测量数据对应路标的索引。

还有一种基于采样的方法，可以得到机器人的位姿：
![](5.png)


以上基于特征的算法都假设特征与路标的关联关系是已知的，对于未知情况，即未知空间的探索，将在定位与地图构建章节展开详细论述。

# 总结
本章关于测量概率的计算，提出了很多种模型。
距离传感器的光线模型是直接基于原始感知数据进行概率计算的。
距离传感器的概率域模型将原始感知数据转换为每个障碍物的全局坐标进行概率计算。
基于关联的感知模型将原始数据转化为局部地图进行概率计算。
基于特征的感知模型对原始数据进行特征提取(如路标)进行概率计算。





