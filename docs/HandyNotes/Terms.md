---
layout: default
title: 术语
description: 随手记录一些知识点以备查
---

Shader resource view (SRV)：着色器资源视图

Unordered access view (UAV)：无序访问视图

无序访问视图 (UAV) 是无序访问资源的视图（可包括缓冲区、纹理和纹理数组，但无需多次采样）。 使用 UAV 可通过多个线程临时进行无序读/写访问。 这意味着此资源类型可由多个线程同时读/写，且不会产生内存冲突。 这种同时访问是通过使用 Atomic Functions（原子函数）来进行的。

Pipeline State Object (PSO): 管线状态对象

[back](./)