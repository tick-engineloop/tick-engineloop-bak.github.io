---
layout: default
title: Linearization of nonlinear depth values in screen space
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

# Perspective projection

<p align="center">
  <img src="../../images/LinearizationOfNonlinearDepthValues-1-PerpectiveFrustumAndNDC.png">
  <p align="center" class="caption"> Perspective Frustum and Normalized Device Coordinates (NDC) </p>
</p>

Note that the eye coordinates are defined in the right-handed coordinate system, but NDC uses the left-handed coordinate system. That is, the camera at the origin is looking along -Z axis in eye space, but it is looking along +Z axis in NDC.

we found all entries of perspective projection matrix. The complete projection matrix $\mathbf{M_{\mathit{projection}}}$ is;

$$
\begin{pmatrix}
\frac{2n}{r-l} & 0 & \frac{r+l}{r-l} & 0    \\
\\
0 & \frac{2n}{t-b} & \frac{t+b}{t-b} & 0    \\
\\
0 & 0 & \frac{-(f+n)}{f-n} & \frac{-2fn}{f-n}   \\
\\
0 & 0 & -1 & 0 \\
\end{pmatrix}
$$

因为齐次坐标在 $w$ 分量为 1.0 时，反映的是笛卡尔坐标位置。假设当 $w_{view}=1.0$ 时顶点在摄像机空间的坐标为 $\mathbf{V_{\mathit{view}}}=(x_{view}, \ y_{view}, \ z_{view}, \ w_{view})$，对应的在裁剪空间和 NDC 中的坐标分别为 $\mathbf{V_{\mathit{clip}}}=(x_{clip}, \ y_{clip}, \ z_{clip}, \ w_{clip})$ 和 $\mathbf{V_{\mathit{ndc}}}=(x_{ndc}, \ y_{ndc}, \ z_{ndc}, \ w_{ndc})$。摄像机空间坐标转换到裁剪空间坐标：

$$
\underbrace{
\begin{pmatrix}
x_{clip}    \\
y_{clip}    \\
z_{clip}    \\
w_{clip}
\end{pmatrix}
}_{\mathbf{V}_{clip}}
=
\underbrace{
\begin{pmatrix}
\frac{2n}{r-l} & 0 & \frac{r+l}{r-l} & 0    \\
\\
0 & \frac{2n}{t-b} & \frac{t+b}{t-b} & 0    \\
\\
0 & 0 & \frac{-(f+n)}{f-n} & \frac{-2fn}{f-n}   \\
\\
0 & 0 & -1 & 0 \\
\end{pmatrix}
}_{\mathbf{M}_{projection}}
\underbrace{
\begin{pmatrix}
x_{view}    \\
y_{view}    \\
z_{view}    \\
w_{view}
\end{pmatrix}
}_{\mathbf{V}_{view}}
$$

> ## References:
>
> * [OpenGL Projection Matrix --- songho](https://www.songho.ca/opengl/gl_projectionmatrix.html)


[back](./)