---
layout: default
title: 面试项目介绍
description: 面试项目要点记录
---

# 态势显示软件

应用中最重要的就是航迹和点迹的显示，也是对性能影响最大的任务。包含两千批航迹和一万批点迹。每批航迹显示要素由一个图标、实体模型、常规标牌、详细标牌和一千历史点组成，常规标牌只有一行显示航迹号，详细标牌可以是多行信息，如航迹号、位置、航向、速度高度、目标属性和目标类别等。每批点迹显示要素由一个图标和一个常规标牌组成。

优化方案包括几个方面：1.图标、模型和点使用实例化渲染，共用顶点数据；2.根据设备和应用长时间待机不需要频繁启动的特点，使用预创建显示要素的方法，创建航迹的一组显示要素后为其设置一个要素集索引，在新的航迹号被接收到后申请可用的要素集索引与要素集关联，根据航迹属性设置要素集中显示要素的属性；3.根据显示要素的动静特征，创建不同类型的显示要素集合，并调整 OpenGL 相应缓存的 BufferUsage。

# 三维渲染引擎产品开发与维护

## 疑难点

### DCE 问题
接口绑定失败，绑定的类未导出到WASM模块中，检查类和绑定实现是否存在错误，查看编译生成WASM模块的文本格式文件WAST，在其中确实未找到绑定的类，在已绑定成功类的源文件中进行绑定测试，可以绑定成功并在WAST中找到绑定信息，结合一些实验，并查阅 Emscripten 官方文档，确定为编译参数(--whole-archive)问题。

### 继承问题
WASM的embind只支持单继承，不支持多继承，Web端需要支持拾取，Primitive就需要同时继承自PrimitiveCollectionBase和PickPrimitive，早期做法是改变继承关系，查阅官方文档和示例代码都没有参考。优化多重继承的方式，最后想到通过embind中的mixin方式，将PickPrimitive的方法绑定到Primitive上，在不改变继承关系的情况下，从而添加了Primitive对拾取功能的支持；

### 内存问题
JS代码中绘制图形鼠标左键按下动作回调函数（JS#StartDrawingCircle#LeftDownAction#Invoke）调用结束后崩溃，通过熟悉WASM指令，看到执行流：
func flow:
5275(EventOption) --> 13683(_free) --> 325(_abort)
触发传参EventOption在胶水文件的runDestructor中_free时引发_abort。

分析malloc块内存模型，以EventOption为例，EventOption对象在共享线性内存中的状态是：
new所创建的EventOption对象存储在类型为Uint8Arry的HEAPU8堆中，所在块大小211byte，数据大小200byte，有4byte的头部和脚部各一个，其余为填充部分。头部和脚部中前29位指示块大小，低三位中用一位表示该块是空闲还是已分配，一般C库中是分配位是最低位，但在JavaScript中尚不确定分配位是哪一位。
根据EventOption内存模型，现在可以解释问题：_free中为何触发_abort，即 func flow: 5275(EventOption) --> 13683(_free) --> 325(_abort):
free中在比较EventOption块头部的低三位时，低三位为1，触发_abort，也就是EventOption块头部的低三位可以为0、2或3，但不能为1。

分析内存chunk结构，堆chunk大小总是8的倍数，所以chunk header高29位记录chunk size，低三位可留作allocated/free或其他信息的标志位。并进行大量实验发现：C++ --> JS 的接口调用中类对象引用类型参数传递传入JS的是一个浅拷贝，而指针类型参数传递传入JS的是对象本身。基于实验发现，在C++ --> JS 的接口调用中，若希望后续传参仍然可以继续使用，我们考虑应使用引用类型参数传递，故目前使用指针类型参数传递的接口调用 C++#ScreenSpaceAction#invoke --> JS#StartDrawingCircle#LeftDownAction#Invoke，应改为使用引用类型参数传递，否则，就会出现EventOption的块头低三位有时为1，使得在释放EventOption时发生崩溃。此外，基于上述实验发现，联系另一 C++ --> JS 的使用引用类型参数传递的接口调用过程，即C++#PrimitiveCollection#update --> JS#ChangeablePrimitive#update，可判定JS#ChangeablePrimitive#update中三个从C++#PrimitiveCollection#update传入的参数context、framestate、commandlist，在JS#ChangeablePrimitive#update调用结束后对参数析构时均被破坏。而在主渲染中context、framestate、commandlist的原始参数仍在使用。上述破坏产生了连锁反映，造成一定内存链表的问题，故context、framestate、commandlist类的析构函数中不进行相关成员释放。

### 疑难问题解决思路

多查阅文档资料，搞懂原理，问题就好解决，资料比较少时就多做实验分析，实在难以解决的会暂时搁置一小段时间，回头再看，以便跳出原来思维，从整体方面进行梳理，发现新的思考角度

## 优化点

### pick

使用颜色缓冲实现拾取。pick 管线和 render 类似，但进行了简化，例如关闭了天空盒、大气和太阳，因为它们是不可拾取的。首先在对象创建更新的时候为每个拾取对象创建了唯一的颜色 ID，将这个 ID 和 拾取对象关联放在 map 中，然后在管线中，根据拾取点位置创建一个剔除体，对渲染对象列表中拾取对象范围之外的其他对象进行剔除，其次使用与之对应的颜色 ID 对渲染对象列表中的对象进行绘制并写入拾取帧缓冲。最后使用 readPixels 读出拾取位置的屏幕颜色，在ID关联拾取对象的关系图中检索，并返回选取的对象。

### Batch

图元集合管理、批汇量绘制：由于性能通常取决于命令的数量，因此许多图元使用批处理通过将不同的对象组合到一个命令中来减少命令的数量。这样也可以减少OpenGL状态切换，使GPU执行在长流水的高效工作状态。例如，BillboardCollection 将尽可能多的广告牌存储在一个顶点缓冲区中，并使用相同的着色器呈现它们。

DMA顶点数据传输：
GPU可使用的存储分为两个部分，一个是显存（VRAM），一个是分配在主存中由GTT(Graphic Translation Table)映射管理的内存，这部分内存即是CPU可访问的又是GPU可访问的。GPU访问显存速度远远大于访问GTT内存速度。
对Batch处理的顶点数据使用DMA传输，利用DMA高效传输的特性，提升主存与显存传输效率，减少CPU等待时间。
创建一个buffer，将其usage设置为GL_STREAM_DRAW，并使用顶点数据进行初始化，这样数据将存储在GTT管理的内存中，再将这个buffer绑定到READ节点，然后创建另一个buffer，按用途设置其usage，将其绑定到WRITE节点，最后使用glCopyBufferSubData进行拷贝。

图元集合设计：期望的设计是将图元集中在少量的Batch集合中，Batch集合越少越好，将静态和动态的分开到不同的Batch集合中，相同更新频率的图元组织在同一个Batch集合中，给其配置不同的 buffer usage。

### OpenGL渲染状态差分式更新

由于当代显卡驱动做了很多工作：状态管理/容错处理/逻辑计算/显存管理等等，属于重度封装，会消耗较多性能。所以，尽可能减少状态切换，是优化渲染性能的重要措施。

### 对数深度

对数深度的z值分布相比于倒数深度更好，不用再分成 1-1000，1000-1000000，1000000-1000000000三个视锥体，只用0.1-1000000000一个视锥体就可以，同时避免了在两个视锥体交界处，同一个物体与两个视锥体同时相交，而在俩个视锥体的绘制命令列表里同时存在，导致重复绘制，增加drawcall数量的问题。

### 实例化


### 地平线剔除、距离剔除与视锥体剔除


### 优化模型

优化纹理：使用gltf-pipeline解包glb，优化纹理大小，再打包成glb，这样在不同场景和距离下使用不同的模型资源
优化网格体：使用meshoptimizer优化GPU渲染管线中的顶点和索引数据，使用gltfpack来优化gltf模型文件

meshoptimizer提供一系列算法来优化GPU渲染管线中的顶点和索引数据。它不仅包含了减少内存占用和提高渲染效率的工具，还提供了简化复杂度的功能。meshoptimizer的优化流程分为7个主要步骤：
索引：通过消除冗余顶点生成有效的索引缓冲。
简化：用于降低模型复杂性。
顶点缓存优化：改变三角形顺序以最大化顶点缓存命中率。
过绘制优化：重新排序三角形，减少像素着色器不必要的执行。
顶点提取优化：优化数据访问模式，提升内存利用率。
顶点量化：将浮点数据转换为更小的数据类型，节省内存。
索引/顶点缓冲压缩：进一步减小存储需求。
该库的一系列高级算法使得在保持图像质量的同时，大幅提高了渲染性能。
取决于场景复杂度和模型复杂度，建模是否合理，通常可提升10%-50%。

### 控制绘制顺序
绘制不透明物体：对不透明物体进行排序，须在相机空间，离相机近的排在前面。绘制不透明物体，离相机近的先绘制。
绘制半透明物体：对半透明物体进行排序，须在相机空间，离相机远的排在前面。绘制半透明物体，离相机远的先绘制。

先绘制不透明物体 pass，再绘制半透明物体 pass。

### 负载均衡
CPU利用率高，GPU利用率低，可以将部分计算如矩阵运算放在GPU中计算，反之将其放在CPU中算

### shader优化
避免if分支，因为着色器调用希望同步运行，这样运行效率最高，为保持同步，if的各分支都会被执行
能在VertexShader中计算的不要放在FragmentShader中，可以在VertexShader中计算，然后利用自动插值获得FragmentShader中的值，因为光栅化后会产生非常多的FragmentShader调用
将跟顶点或像素无关的简单计算移到CPU，然后通过uniform传进来。

### oit

在绘制半透明物体时，通常需要对其进行排序，先绘制远处的物体，再绘制近处的物体。然而复杂的场景和几何模型都会加重CPU排序的难度和工作量，可使用oit，避免CPU的排序工作。oit有精确oit和近似oit，偏效果的高性能设备上可使用精确oit（每像素链表），偏性能的性能受限设备上可使用近似oit（权重混合）。


# Advanced Lighting

## Shadow Map

Cascaded Shadow Map
Native -> PCF(Percentage Close Filtering) -> PCSS(Percentage Close Soft Shadow) -> VSSM(Variance Soft Shadow Mapping)

## SSAO

SSAO(Screen Space Ambient Occlusion) -> HBAO(Horizon Based Ambient Occlusion) -> SSDO(Screen Space Directional Occlusion) -> SSR(Screen Space Reflection)

## PBR

PBR 遵循使用 cook-torrance BRDF 的反射方程

材质模型/工作流：SG(Specular Glossiness)、MR(Metallic Roughness)


# Graphics API

## OpenGL 和 OpenGL ES 

OpenGL 是一个功能全面的图形渲染API，提供了丰富的API调用，支持复杂的图形渲染和高级特性，适用于需要高性能图形处理的桌面和服务器应用；而 OpenGL ES 则是针对嵌入式移动设备设计的精简版 OpenGL，是 OpenGL 的子集，功能相对简化，主要针对嵌入式设备的性能限制进行了优化，通过优化和裁剪，提升在移动设备上的渲染性能和兼容性。

图元类型：OpenGL支持更广泛的图元类型，而OpenGL ES通常限于点、线和三角形等基本类型。
扩展和可编程性：支持的扩展不同。由于OpenGL ES的目标是保持简洁和高效，因此它可能不支持某些OpenGL的扩展。

## Vulkan 和 OpenGL

Vulkan 和 OpenGL 在多个方面存在显著差异，这些差异主要体现在设计目标、性能、功能、易用性等方面。以下是对两者区别的详细归纳和优势分析：

Vulkan 与 OpenGL的区别
1. 设计目标
Vulkan：旨在提供更低的驱动程序开销和更高的性能，同时支持跨平台图形。它假设程序员对图形硬件的行为有深入的了解，提供了更详细的硬件控制和更高效的资源管理。
OpenGL：提供了更高层次的抽象，使得开发者可以更容易地创建图形应用程序。它隐藏了许多底层细节，简化了编程，但可能牺牲了部分性能。
2. 性能与开销
Vulkan：具有更低的驱动程序开销，意味着更少的CPU时间被用于图形调用，从而提高性能。它提供了更高效的内存管理机制和更灵活的管线状态管理，允许开发者更好地优化应用程序性能。驱动程序没有运行时错误检测，但存在针对开发人员的验证层。
OpenGL：驱动程序开销相对较高，特别是在高负载的情况下，可能会成为性能瓶颈。它缺乏对底层硬件的直接控制，可能影响性能优化。存在广泛的运行时错误检测。
3. 编程模型
Vulkan：是一个显式API，要求开发者明确管理图形硬件的许多方面，如内存管理和同步。它引入了固定的图形管线结构，要求开发者在开始渲染前设置好所有的状态。
OpenGL：相对隐式，许多底层细节对开发者来说是不透明的。它允许更动态的管线，可以在渲染过程中改变更多的状态。
4. 多线程支持
Vulkan：支持多线程并行处理，允许在多个线程中创建和管理资源，提高了并行性和效率。
OpenGL：对多线程的支持较差，许多操作不能在多个线程中安全执行。
5. 跨平台性
Vulkan：旨在支持跨平台图形，可以在多种操作系统和硬件上运行，包括Windows、Linux、Android和macOS。
OpenGL：主要用于桌面和服务器环境，有针对移动设备的OpenGL ES版本。
6. 状态机
Vulkan:基于实例的状态
OpenGL：全局状态机

Vulkan 的优势
更低的驱动程序开销：提高了性能，减少了CPU在图形调用上的时间消耗。
更高效的内存管理：使图形开发者能够更方便地控制图形渲染流程和优化内存使用。
灵活的管线状态管理：允许开发者在渲染过程中更好地控制图形渲染流程。
多线程支持：支持在多个CPU核心上分配工作，提高了并行性和效率。
跨平台性：支持多种操作系统和硬件平台，具有广泛的兼容性。
强大的错误检查和调试工具：通过验证层提供了详细的错误检查和调试信息，帮助开发者发现和修复问题。

OpenGL 的优势
广泛的支持：在许多平台和旧有设备上都有广泛的支持，包括Android 7.0以下版本的硬件设备。
丰富的教程和学习资源：有大量现有的教程和学习资源可供学习和参考，学习曲线较为平缓。
易用性：提供了更高层次的抽象，简化了编程，使得开发者可以更容易地创建图形应用程序。

综上所述，Vulkan 和 OpenGL 各有其优势和适用场景。Vulkan 更适合需要高性能和更底层控制的应用程序开发，而 OpenGL 则更适合需要更成熟的生态系统支持或面向设备为较旧的硬件设备的应用程序开发。

# 移动端优化经验                       


# ECS



# 抗锯齿

## FXAA

## MSAA