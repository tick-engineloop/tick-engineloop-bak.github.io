---
layout: default
title: 计算机图形学编程语言随手记
description: 随手记录一些小的知识点
---

OpenGL 图形管线执行流程（简化版）：

![OpenGL 图形管线执行流程](../../images/GraphicPipeline-OpenGL.png)

Vulkan 图形管线执行流程：

![Vulkan 图形管线执行流程](../../images/GraphicPipeline-Vulkan.svg)

可以看到，图元装配是按照指定的拓扑将一个类似于 glDrawElements 这样的绘制指令涉及的一系列顶点组装成为目标形状的操作，OpenGL 的输入装配是在顶点着色器和片段着色器之间，而 Vulkan 的输入装配是在顶点着色器之前；OpenGL 的模版与深度测试默认是在【测试与混合】阶段执行，在后期 OpenGL 还加入了 Early Fragment Test 特性，允许模版与深度等测试在片段着色器之前执行。Vulkan 在片段着色器前后有一个 Early Per-Fragment Test 和 Late Per-Fragment Test, 可以指定模版与深度测试在哪个阶段执行，默认是在 Late Per-Fragment Test 阶段执行。