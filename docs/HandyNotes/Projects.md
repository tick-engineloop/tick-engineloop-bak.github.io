---
layout: default
title: 面试项目介绍
description: 面试项目要点记录
---

# 态势显示软件

应用中最重要的就是航迹和点迹的显示，也是对性能影响最大的任务。包含两千批航迹和一万批点迹。每批航迹显示要素由一个图标、实体模型、常规标牌、详细标牌和一千历史点组成，常规标牌只有一行显示航迹号，详细标牌可以是多行显示航迹号、位置、航向、速度高度、目标属性和目标类别等。每批点迹显示要素由一个图标和一个常规标牌组成。优化方案包括几个方面：1.图标、模型和点使用实例化渲染，共用顶点数据；2.根据设备和应用长时间待机不需要频繁启动的特点，使用预创建显示要素的方法，创建航迹的一组显示要素后为期设置一个要素集索引，在新的航迹号被接收到后申请可用的要素集索引与要素集关联，根据航迹属性设置要素集中显示要素的属性；3.根据显示要素的动静特征，创建不同类型的显示要素集合，并调整 OpenGL 相应缓存的 BufferUsage。

# 三维渲染引擎产品开发与维护

疑难问题：DCE导致的接口绑定失败，绑定的类未导出到WASM模块中，检查类和绑定实现是否存在错误，通过编译生成WASM模块的文本格式文件WAST，在其中确实未找到绑定的类，在已绑定成功类的源文件中进行绑定测试，可以绑定成功并在WAST中找到绑定信息，通过各种实验并查阅 Emscripten 官方文档，确定为编译参数(--whole-archive)问题。

WASM的embind只支持单继承，不支持多继承，Primitive同时继承自PrimitiveCollectionBase和PickPrimitive，Web端需要支持拾取，早期做法是改变继承关系，后来查阅官方文档和示例代码，优化多重继承的方式，想到通过embind中的mixin方式，将PickPrimitive的方法绑定到Primitive上，在不改变继承关系的情况下，从而添加了Primitive对拾取功能的支持；

JS代码中绘制图形鼠标左键按下动作回调函数（JS#StartDrawingCircle#LeftDownAction#Invoke）调用结束后崩溃，通过熟悉WASM指令，看到执行流：
func flow:
5275(EventOption) --> 13683(_free) --> 325(_abort)
触发传参EventOption在胶水文件的runDestructor中_free时引发_abort。

分析malloc块内存模型，以EventOption为例，EventOption对象在共享线性内存中的状态是：
new所创建的EventOption对象存储在类型为Uint8Arry的HEAPU8堆中，所在块大小211byte，数据大小200byte，有4byte的头部和脚部各一个，其余为填充部分。头部和脚部中前29位指示块大小，低三位中用一位表示该块是空闲还是已分配，一般C库中是分配位是最低位，但在JavaScript中尚不确定分配位是哪一位。
根据EventOption内存模型，现在可以解释问题：_free中为何触发_abort，即 func flow: 5275(EventOption) --> 13683(_free) --> 325(_abort):
free中在比较EventOption块头部的低三位时，低三位为1，触发_abort，也就是EventOption块头部的低三位可以为0、2或3，但不能为1。

分析内存chunk结构，堆chunk大小总是8的倍数，所以chunk header高29位记录chunk size，低三位可留作allocated/free或其他信息的标志位。并进行大量实验发现：C++ --> JS 的接口调用中类对象引用类型参数传递传入JS的是一个浅拷贝，而指针类型参数传递传入JS的是对象本身。基于实验发现，在C++ --> JS 的接口调用中，若希望后续传参仍然可以继续使用，我们考虑应使用引用类型参数传递，故目前使用指针类型参数传递的接口调用 C++#ScreenSpaceAction#invoke --> JS#StartDrawingCircle#LeftDownAction#Invoke，应改为使用引用类型参数传递，否则，就会出现EventOption的块头低三位有时为1，使得在释放EventOption时发生崩溃。此外，基于上述实验发现，联系另一 C++ --> JS 的使用引用类型参数传递的接口调用过程，即C++#PrimitiveCollection#update --> JS#ChangeablePrimitive#update，可判定JS#ChangeablePrimitive#update中三个从C++#PrimitiveCollection#update传入的参数context、framestate、commandlist，在JS#ChangeablePrimitive#update调用结束后对参数析构时均被破坏。而在主渲染中context、framestate、commandlist的原始参数仍在使用。上述破坏产生了连锁反映，造成一定内存链表的问题，故context、framestate、commandlist类的析构函数中不进行相关成员释放。