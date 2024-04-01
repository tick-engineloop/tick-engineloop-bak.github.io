---
layout: default
title: 法向量变换
description: 法向量变换矩阵推导
---

&emsp;&emsp;多边形模型中的顶点除了空间位置信息，还包括一些关于该顶点与周围表面关系的附加信息。在所有附加信息中，切向量和法向量是最常见的顶点附加信息，当进行模型变换时，不仅要变换顶点的位置，还要变换这些法向量和切向量。

&emsp;&emsp;切向量可通过计算一个顶点向量和另外一个顶点向量之间的差获得，因此可以用变换后的两个顶点向量之间的差表示变换后的切向量。如果在顶点变换中使用了 3x3 矩阵 $\bold{M}$，那么该顶点的切向量也可用矩阵 $\bold{M}$ 进行变换，由于切向量和法向量不受平移变换的影响，这里的变换采用 3x3 矩阵 $\bold{M}$。

&emsp;&emsp;与切向量的变换相比，法向量的变换要复杂一些。下图是一个带有法向量 $\bold{T}$ 和切向量 $\bold{N}$ 的三角形：

<center>

![image](../../images/NormalTransform1.png)

</center>

当用一个包含非均匀缩放的非正交矩阵 $\bold{M}$ 变换法向量时，变换后的法向量常常指向一个与变换表面不垂直的方向，如下图所示：

<center>

![image](../../images/NormalTransform2.png)

</center>

&emsp;&emsp;因为切向量和法向量总是垂直的，则同一个顶点的切向量 $\bold{T}$ 和法向量 $\bold{N}$ 一定满足方程 $\bold{T} \cdot \bold{N}=0$，变换后的切向量 $\bold{T}^{\prime}$ 和法向量 $\bold{N}^{\prime}$ 也满足该方程，给定一个变换矩阵 $\bold{M}$，$\bold{T}^{\prime}=\bold{MT}$。假设法向量 $\bold{N}$ 的变换矩阵为 $\bold{G}$，则下式成立：

$\bold{N}^{\prime} \cdot \bold{T}^{\prime}=(\bold{G}\bold{N}) \cdot (\bold{MT})$=0

经代数变换可得：

$(\bold{G}\bold{N}) \cdot (\bold{MT})=(\bold{G}\bold{N})^{\bold{T}}(\bold{MT})=\bold{N}^{\bold{T}}\bold{G}^{\bold{T}}\bold{MT}$

&emsp;&emsp;由于 $\bold{N}^{\bold{T}}\bold{T}=0$，如果 $\bold{G}^{\bold{T}}\bold{M}=\bold{I}$，则 $\bold{N}^{\bold{T}}\bold{G}^{\bold{T}}\bold{MT}=0$，可知 $\bold{G}=(\bold{M}^{-1})^{\bold{T}}$。因此，使法向量正确旋转的变换矩阵应为顶点变换矩阵的逆矩阵的转置矩阵。

&emsp;&emsp;如果矩阵 $\bold{M}$ 为正交矩阵，那么 $\bold{M}^{-1}=\bold{M}^{\bold{T}}$，则 $(\bold{M}^{-1})^{\bold{T}}=\bold{M}$，这时为了计算法向量的变换矩阵的求逆矩阵和转置矩阵的操作可以避免。

