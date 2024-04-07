---
layout: default
title: 计算着色器知识点记录
description: 随手记录一些小的知识点
---

# Compute Shader

## Compute space

The "space" that a compute shader operates on is largely abstract. There is the concept of a work group, this is the smallest amount of compute operations that the user can execute. The number of work groups that a compute operation is executed with is defined by the user when they invoke the compute operation.

计算着色器操作的空间是极其抽象的。在这里有一个工作组的概念，它是用户可以执行的计算操作的最小单元。执行计算操作的工作组数由用户在调用计算操作时定义。

When the system actually computes the work groups, it can do so in any order. So if it is given a work group set of (3, 1, 2), it could execute group (0, 0, 0) first, then skip to group (1, 0, 1), then jump to (2, 0, 0), etc. So your compute shader should not rely on the order in which individual groups are processed.

当系统实际计算工作组时，它可以按任何顺序进行计算。因此，如果给它一个 （3， 1， 2） 的工作组集，它可以先执行组 （0， 0， 0），然后跳到组 （1， 0， 1），然后跳转到 （2， 0， 0） 等。因此，计算着色器不应依赖于处理各个组的顺序。

Do not think that a single work group is the same thing as a single compute shader invocation; there's a reason why it is called a "group". Within a single work group, there may be many compute shader invocations. How many is defined by the compute shader itself, This is known as the local size of the work group.

不要认为单个工作组与单个计算着色器调用是一回事；它被称为“组”是有原因的。在单个工作组中，可能有许多计算着色器调用。多少由计算着色器本身定义，这称为工作组的本地大小。

Every compute shader has a three-dimensional local size. This defines the number of invocations of the shader that will take place within each work group.

每个计算着色器都有一个三维局部大小。这定义了将在每个工作组中发生的着色器调用次数。

The individual invocations within a work group will be executed "in parallel". The main purpose of the distinction between work group count and local size is that the different compute shader invocations within a work group can communicate through a set of shared variables and special functions. Invocations in different work groups (within the same compute shader dispatch) cannot effectively communicate. 

工作组内的单个调用将“并行”执行。区分工作组数量和本地大小的主要目的是，工作组中的不同计算着色器调用可以通过一组共享变量和特殊函数进行通信。不同工作组中的调用（在同一计算着色器调度中）无法有效通信。

## Inputs

Compute shaders cannot have any user-defined input variables. If you wish to provide input to a CS, you must use the implementation-defined inputs coupled with resources like storage buffers or Textures. 

计算着色器不能有任何用户定义的输入变量。如果要向 CS 提供输入，则必须使用与资源（像存储缓冲区或纹理等）相结合的实现定义的输入。

Compute Shaders have the following built-in input variables.

计算着色器有以下内置输入变量。


```glsl
// In the compute language, gl_NumWorkGroups contains the total number of work groups that will execute the compute shader. 
// The components of gl_NumWorkGroups are equal to the num_groups_x, num_groups_y, and num_groups_z parameters passed to the glDispatchCompute command.
in uvec3 gl_NumWorkGroups ;         // 工作组的总数量

// In the compute language, gl_WorkGroupSize contains the size of a workgroup declared by a compute shader. 
// The size of the work group in the X, Y, and Z dimensions is stored in the x, y, and z components of gl_WorkGroupSize. 
// The values stored in gl_WorkGroupSize match those specified in the required local_size_x, local_size_y, and local_size_z layout qualifiers for the current shader. 
// This value is constant so that it can be used to size arrays of memory that can be shared within the local work group.
const uvec3 gl_WorkGroupSize ;      // 由一个计算着色器声明的一个工作组的大小

// In the compute language, gl_WorkGroupID contains the 3-dimensional index of the global work group that the current compute shader invocation is executing within. 
// The possible values range across the parameters passed into glDispatchCompute, i.e., from (0, 0, 0) to (gl_NumWorkGroups.x - 1, gl_NumWorkGroups.y - 1, gl_NumWorkGroups.z - 1).
in uvec3 gl_WorkGroupID ;           // 当前计算着色器调用正执行在索引为 gl_WorkGroupID 的全局工作组内

// In the compute language, gl_LocalInvocationID is an input variable containing the n-dimensional index of the local work invocation within the work group that the current shader is executing in. 
// The possible values for this variable range across the local work group size, i.e., (0,0,0) to (gl_WorkGroupSize.x - 1, gl_WorkGroupSize.y - 1, gl_WorkGroupSize.z - 1).
in uvec3 gl_LocalInvocationID ;     // 当前计算着色器正执行在索引为 gl_LocalInvocationID 的局部工作调用内

// In the compute language, gl_GlobalInvocationID is a derived input variable containing the n-dimensional index of the work invocation within the global work group that the current shader is executing on. 
// The value of gl_GlobalInvocationID is equal to gl_WorkGroupID * gl_WorkGroupSize + gl_LocalInvocationID.
in uvec3 gl_GlobalInvocationID ;    // 当前计算着色器调用正执行在全局工作组内的索引为 gl_GlobalInvocationID 的工作调用上
```