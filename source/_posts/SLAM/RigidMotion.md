---
title: 三维空间中的刚体运动
date: 2017-02-24 10:11:25
tags: 
- SLAM基础 
- 刚体运动
categories:
- 机器人事业
- SLAM
description: 关于学习三维空间中刚体运动的简单记录。
---
<!-- more -->
# 摘要
SLAM中一个很基本的问题就是计算机器人在三维空间中的位姿。位姿包括两个部分，第一是位置，其次是姿态。而机器人在三维空间中位姿的计算往往与三维空间中的刚体运动有关。
这篇博客是关于学习三维空间中如何表示位姿的简单记录。

# 位置表示
在一个三维空间中，建立三维坐标系之后，就可以用一个三维坐标来表示机器人的位置。
对于机器人的位置变换，可以用一个三维的平移向量来表示，比如机器人从初始位置为$(x\_0, y\_0, z\_0)$经过一个平移向量$(a,b,c)$，可以得到平移后的位置$(x\_0+a, y\_0+b, z\_0+c)$。相对来说，机器人的位置表达比较简单。


# 姿态表示
在一个三维空间中，通常用一个三维向量来表示机器人的姿态。更直观的讲，机器人的姿态可以想象成机器人自带一个坐标系，这个坐标系的原点就是机器人，z轴表示机器人面向的方向，这样的一个坐标系可以通过表示机器人姿态的三维向量来构造。
对于机器人的姿态则相对复杂。通常有欧拉角，四元组，旋转向量，旋转矩阵等。

# 旋转矩阵
在SLAM问题中，旋转矩阵是表示姿态变换最常用的方式。
假设空间中的某一点$P$，机器人旋转前的坐标系的单位正交基为$(e\_1,e\_2,e\_3)$，$P$的坐标为$(x\_1,y\_1,z\_1)$。机器人旋转后的坐标系的单位正交基为$(e'\_1,e'\_2,e'\_3 )$，$P$的坐标为$(x'\_1,y'\_1,z'\_1)$。
根据坐标的定义有：

$\begin{bmatrix}
e\_1 & e\_2 & e\_3
\end{bmatrix}\begin{bmatrix}
a\_1\\\ 
a\_2\\\
a\_3
\end{bmatrix}=\begin{bmatrix}
e'\_1 & e'\_2 & e'\_3
\end{bmatrix}\begin{bmatrix}
a'\_1\\\
a'\_2\\\ 
a'\_3
\end{bmatrix}$

在等式的左右同时左乘$\begin{bmatrix}
a\_1^T\\\ 
a\_2^T\\\
a\_3^T
\end{bmatrix}$，可得

$\begin{bmatrix}
a\_1\\\ 
a\_2\\\
a\_3
\end{bmatrix}=\begin{bmatrix}
e\_1^T e'\_1 & e\_1^T e'\_2 & e\_1^T e'\_3 \\\
e\_2^T e'\_1 & e\_2^T e'\_2 & e\_2^T e'\_3 \\\
e\_3^T e'\_1 & e\_3^T e'\_2 & e\_3^ Te'\_3
\end{bmatrix}\begin{bmatrix}
a'\_1\\\
a'\_2\\\ 
a'\_3
\end{bmatrix}=Ra'$

矩阵$R$即为旋转矩阵。旋转矩阵是一个行列式为1的正交矩阵。同样所有行列式为1的正交矩阵都是旋转矩阵，因此可以把旋转矩阵做如下定义：
$SO(n)=\\{R\epsilon \mathbb{R}^{n\times n} \mid RR^T=I,det(R)=1\\}$
$SO(n)$称为特殊正交群，这个集合由n维空间的旋转矩阵组成，特别的，SO(3)就是三维空间下的旋转矩阵。

值得注意的是，旋转矩阵本质上是表示两个坐标系之间的旋转，而坐标系恰恰能够表示机器人姿态，因此旋转矩阵也可以描述机器人的旋转，即姿态变换。

# 变换矩阵
在旋转矩阵的基础之上，加上平移向量，就可以完整的刻画三维空间中的刚体运动，即：
$a'=Ra+t$

但是这种形式下会存在一个问题，假设我们进行了两次变换$R\_1$,$t\_1$和$R\_2$,$t\_2$。相应的三维空间中经历了从a点到b点到c点的变换。则满足公式：
$b=R\_1a+t\_1, c=R\_2b+t\_2$

从a到c的变换为：
$c=R\_2(R\_1a+t\_1)+t\_2$

这样的形式在变换多次之后会过于复杂。聪明的数学家引入了齐次坐标和变换矩阵的概念，使得三维空间中的刚体运动有如下的变换形式：

$\begin{bmatrix}
a'\\\ 
1
\end{bmatrix}=\begin{bmatrix}
R & t \\\ 
0^T & 1  
\end{bmatrix}\begin{bmatrix}
a\\\
1
\end{bmatrix} = T\begin{bmatrix}
a\\\
1
\end{bmatrix}$

矩阵T即称为变换矩阵

此时从a到c的变换可表示为：
$c=T\_2T\_1a$
此处的a和c为相应齐次坐标。

变换矩阵的左上角为旋转矩阵，右侧为平移向量，左下角为0向量，右下角为1。这种矩阵称为特殊欧式群：

$SE(3)=\\{T=\begin{bmatrix}
R & t \\\ 
0^T & 1 
\end{bmatrix} \epsilon \mathbb{R}^{4 \times 4} \mid R \epsilon SO(3), t\epsilon \mathbb{R}^3 \\}$

# 旋转向量
用旋转矩阵来表示旋转有两个缺点：
1.$SO(3)$的旋转矩阵有九个量，但一次旋转只有三个自由度。因此这种表达方式是冗余的
2.旋转矩阵自身两个约束，即必须是正交矩阵和行列式为1，这些约束会使得求解变得更困难。

从直观上讲，任意旋转都可以用一个旋转轴和一个旋转角来刻画。于是可以使用一个向量，其方向与旋转轴一致，而长度等于旋转角。这种向量称为旋转向量，只需要一个三维向量就可以描述旋转。旋转向量跟之后要介绍的李代数是相对应的。旋转向量到旋转矩阵的变换可由**罗德里格斯公式**求得。
假设旋转轴为$n$，角度为$\theta$，则旋转向量为$\theta n$对应的旋转矩阵$R$为：
$R=\cos \theta {I} + (1-\cos \theta)nn^T + \sin \theta [n]\_{\times}$

反之有：
$\theta = \arccos(\dfrac{tr(R)-1}{2})$
由于旋转轴上的向量在旋转后不发生改变，说明：
$Rn = n$
因此转轴$n$是矩阵$R$特征值1对应的特征向量。求解此方程，再归一化，就得到了旋转轴。



# 欧拉角
欧拉角是一种最为直观的姿态变换描述方式。在欧拉角的表示方式中，将旋转分解成沿三个坐标轴旋转的量：滚转角－俯仰角－偏航角(roll-pitch-yaw)。欧拉角的一个重大缺点就是万向锁问题：在俯仰角为$\pm90$度时，第一次旋转与第三次旋转将使用同一个轴，这被称为奇异性问题。
由于欧拉角不适于插值和迭代，所以在SLAM问题中通常不使用欧拉角来表示旋转，所以不再展开记录。


# 四元数
旋转矩阵具有冗余性，欧拉角和旋转向量不冗余但是具有奇异性。
接下来出场的四元数就厉害了，只有四个自由度而且也没有奇异性。它是一种类似于复数的表达方式。
四元数内容较多，另外再单独记录。


# 参考文献
1. 《视觉SLAM十四讲》第三讲

