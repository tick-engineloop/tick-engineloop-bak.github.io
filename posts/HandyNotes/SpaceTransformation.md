---
layout: default
title: 空间变换
description: world space、camera space、clip space、normalized device space. etc
---

<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
                skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
                inlineMath: [['$','$']]
            }
        });
    </script>
</head>

# Homogeneous coordinates

三维笛卡尔坐标位置可以通过三维向量与 3x3 矩阵的乘法操作，来完成缩放和旋转的线性变换。但是平移操作是无法通过与 3x3 矩阵的乘法操作来完成的，因为线性变换总是将 (0, 0, 0) 映射到 (0, 0, 0)。一个可选的处理方式是进行一个额外的仿射变换(affine transformation)操作，将点 (0, 0, 0) 移动到另一个位置。但是加入这个额外的操作后意味着我们将无法再运用线性变换的各种特性，如将多个变换过程合并为一次变换。因此，我们需要找到一种方法，通过使用线性变换来表达平移过程。幸运的是，只要将三维空间坐标提升一个维度置入到四维空间当中，仿射变换就回归成为一种简单的线性变换了，也就是说我们可以直接使用 4x4 矩阵的乘法来完成模型的移动操作了。

举例来说，假设位置向量第四个分量为 1.0，将位置沿 y 轴移动 0.3，则有：

$$
\begin{bmatrix}
1.0 & 0.0 & 0.0 & 0.0 \\
0.0 & 1.0 & 0.0 & 0.3 \\
0.0 & 0.0 & 1.0 & 0.0 \\
0.0 & 0.0 & 0.0 & 1.0 \\
\end{bmatrix}

\begin{pmatrix}
x   \\
y   \\
z   \\
1.0 \\
\end{pmatrix}

=

\begin{pmatrix}
x       \\
y + 0.3 \\
z       \\
1.0     \\
\end{pmatrix}
$$

所谓齐次坐标就是将一个原本是 n 维的向量用一个 n+1 维向量来表示。对于二维或三维的笛卡尔坐标点我们可以直接添加第四个分量（通常以符号 $w$ 标识），并设置值为 1.0 来实现齐次坐标的建立。

$$
(2.0, \ 3.0, \ 5.0) \ \to \ (2.0, \ 3.0, \ 5.0, \ 1.0)
$$

当我们获得齐次坐标后，可以使用第四个分量除以所有的分量，并将其舍弃，以重新得到笛卡尔坐标。

$$
(4.0, \ 6.0, \ 10.0, \ 2.0) \ \overset{\mathsf{divide \ by} \ w}{\longrightarrow} \ (2.0, \ 3.0, \ 5.0, \ 1.0) \overset{\mathsf{drop} \ w}{\longrightarrow} \ (2.0, \ 3.0, \ 5.0)
$$

齐次坐标的第四个分量其实是用来实现透视投影变换的，并且如果所有的分量都除以一个相同的值，那么将不会改变它所表达的坐标位置。

举例来说，以下所有的坐标都表示同一个点：(2.0, 3.0, 5.0, 1.0)、(4.0, 6.0, 10.0, 2.0)、(0.2, 0.3, 0.5, 0.1)。所以，齐次坐标所表示的其实是方向而不是位置。

平移、旋转、缩放变换都不会改变 $w$ 的值。但透视投影变换会将 $w$ 分量修改为 1.0 以外的值。

## Orthographic Projection

正射投影矩阵定义了一个类似立方体的平截头体，它定义了一个裁剪空间，由宽、高、近(Near)平面和远(Far)平面所指定，在这空间之外的顶点都会被裁剪掉。

<p align="center">
  <img src="../../images/SpaceTransformation-OrthographicFrustum.png">
</p>

正射平截头体直接将平截头体内部的所有坐标映射为标准化设备坐标，因为每个向量的 $w$ 分量都不会被改变，如果 $w$ 分量等于1.0，那么透视除法就不会改变这个坐标的值。

## Perspective Projection

实际生活中近大远小的视觉现象被称之为透视。下面定义了一个四棱锥的平截头体，它由视野(fov)、宽高比、近(Near)平面和远(Far)平面所指定。

<p align="center">
  <img src="../../images/SpaceTransformation-PerspectiveFrustum.png">
</p>

这种近大远小的效果可使用透视矩阵来完成。这个投影矩阵将给定的平截头体范围映射到裁剪空间，除此之外还修改了每个顶点坐标的 $w$ 分量，从而使得离观察者越远的顶点坐标 $w$ 分量越大。被变换到裁剪空间的坐标都会在 $-w$ 到 $w$ 的范围之间（任何大于这个范围的坐标都会被裁剪掉）。顶点坐标的每个分量都会除以它的 $w$ 分量，距离观察者越远顶点坐标就会越小。这是也是w分量非常重要的另一个原因，它能够帮助我们进行透视投影。最后的结果坐标就是处于标准化设备空间中的。

# Forword

假设已知投影矩阵 $M_{projection}$、视图矩阵$M_{view}$、模型矩阵$M_{model}$和局部空间坐标 $V_{local}$，一个顶点坐标将会根据以下过程被变换到裁剪坐标：

$$
V_{clip} = M_{projection} \cdot M_{view} \cdot M_{model} \cdot V_{local}
$$

注意矩阵运算不满足交换律，$V_{local}$ 应依次和 $M_{model}$、$M_{view}$、$M_{projection}$ 相乘。顶点着色器的输出要求所有的顶点都在裁剪空间内，最后的顶点应该被赋值到顶点着色器中的 gl_Position，OpenGL将会自动进行裁剪、透视除法和视口变换。

# Backword

## 从 NDC 坐标转换为摄像机空间坐标

因为齐次坐标在 $w$ 分量为 1.0 时，反映的是笛卡尔坐标位置。假设当 $w_{view}=1.0$ 时顶点在摄像机空间的坐标为 $V_{view}=(x_{view}, \ y_{view}, \ z_{view}, \ w_{view})$。对应的在裁剪空间和 NDC 中的坐标分别为 $V_{clip}=(x_{clip}, \ y_{clip}, \ z_{clip}, \ w_{clip})$ 和 $V_{ndc}=(x_{ndc}, \ y_{ndc}, \ z_{ndc}, \ w_{ndc})$。摄像机空间坐标与 NDC 坐标关系为：

$$
V_{ndc} = \frac{V_{clip}}{w_{clip}} \tag{1}
$$

$$
V_{clip} = M_{projection} \cdot V_{view} \tag{2}
$$

由式 (1) 可求得 $V_{clip}$：

$$
V_{clip} = w_{clip} \cdot V_{ndc} \tag{3}
$$

由式 (2) 可求得 $V_{view}$：

$$
V_{view} = M_{projection}^{-1} \cdot V_{clip} \tag{4}
$$

将式 (3) 代入式 (4) 可求得 $V_{view}$：

$$
V_{view} =  M_{projection}^{-1} \cdot (w_{clip} \cdot V_{ndc}) \tag{5}
$$

将 $w_{clip}$ 移到矩阵乘的前面：

$$
V_{view} =  w_{clip} \cdot M_{projection}^{-1} \cdot V_{ndc} \tag{6}
$$

式 (6) 中只有 $w_{clip}$ 是未知的，所以接下来让我们来求取它。这里对于式 (6) 我们可以只观察其中的 $w$ 分量：

$$
w_{view} =  w_{clip} \cdot w_{M_{projection}^{-1} \cdot V_{ndc}} \tag{7}
$$

因为我们在开头假设了 $w_{view}=1.0$，所以由式 (7) 有：

$$
w_{clip} \cdot w_{M_{projection}^{-1} \cdot V_{ndc}} = 1.0 \tag{8}
$$

进而求得 $w_{clip}$：

$$
w_{clip} = \frac{1.0}{w_{M_{projection}^{-1} \cdot V_{ndc}}} \tag{9}
$$

将式 (9) 代入式 (7)，最终可求得 $V_{view}$：

$$
V_{view} =  \frac{1.0}{w_{M_{projection}^{-1} \cdot V_{ndc}}} \cdot M_{projection}^{-1} \cdot V_{ndc} \tag{10}
$$