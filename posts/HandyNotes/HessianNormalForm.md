---
layout: default
title: Hessian Normal Form
description: 平面方程的海森法线形式
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

It is especially convenient to specify planes in so-called Hessian normal form. This is obtained from the general equation of a plane

以所谓的 Hessian 法线形式指定平面尤其方便。这是从平面的一般方程中得到的

$$
ax + by + cz + d = 0    \tag{1}
$$

by defining the components of the unit normal vector $\mathbf{n} = (n_x,\ n_y,\ n_z)$:

通过定义单位法向量的分量 $\mathbf{n} = (n_x,\ n_y,\ n_z)$：

$$
n_x = \frac{a}{\sqrt{a^2+b^2+c^2}}  \tag{2}
$$

$$
n_y = \frac{b}{\sqrt{a^2+b^2+c^2}}  \tag{3}
$$

$$
n_z = \frac{c}{\sqrt{a^2+b^2+c^2}}  \tag{4}
$$

and the constant

和常量

$$
p = \frac{d}{\sqrt{a^2+b^2+c^2}}    \tag{5}
$$

Then the Hessian normal form of the plane is

然后平面的海森法线形式是

$$
\mathbf{n} \cdot \mathbf{x} = -p    \tag{6}
$$

and $p$ is the distance of the plane from the origin. Here, the sign of $p$ determines the side of the plane on which the origin is located. If $p>0$, it is in the half-space determined by the direction of $\mathbf{n}$, and if $p<0$, it is in the other half-space.

并且 $p$ 是从原点到平面的距离。$p$ 的符号决定了原点是位于平面的哪一边。如果 $p>0$，原点是在 $\mathbf{n}$ 的方向确定的一半空间内，如果 $p<0$，原点是在另一半空间内。

The [point-plane distance](https://mathworld.wolfram.com/Point-PlaneDistance.html) from a point $x_0$ to a plane (6) is given by the simple equation

从一点 $x_0$ 到平面 (6) 的点-面距离由简单的方程给出

$$
D = \mathbf{n} \cdot x_0 + p    \tag{7}
$$

If the point $x_0$ is in the half-space determined by the direction of $\mathbf{n}$, then $D>0$; if it is in the other half-space, then $D<0$.

如果点 $x_0$ 在 $\mathbf{n}$ 的方向确定的一半空间里，那么 $D>0$；如果它在另一半空间里，那么 $D<0$。

> ## References:
>
> * [Plane --- Wolfram Mathworld](https://mathworld.wolfram.com/Plane.html)
>
> * [Hessian Normal Form --- Wolfram Mathworld](https://mathworld.wolfram.com/HessianNormalForm.html)
>
> * [Point-Plane Distance --- Wolfram Mathworld](https://mathworld.wolfram.com/Point-PlaneDistance.html)
>

[back](./)