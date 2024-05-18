---
layout: default
title: Deferred Shading
description: 延迟着色
---

# Introduction

The way we did lighting so far was called forward rendering or forward shading. A straightforward approach where we render an object and light it according to all light sources in a scene. We do this for every object individually for each object in the scene. While quite easy to understand and implement it is also quite heavy on performance as each rendered object has to iterate over each light source for every rendered fragment, which is a lot! Forward rendering also tends to waste a lot of fragment shader runs in scenes with a high depth complexity (multiple objects cover the same screen pixel) as fragment shader outputs are overwritten.

到目前为止，我们实现光照的方式被称为前向渲染或前向着色。它是一种非常直接的方式，我们渲染一个对象，并使用场景中的所有光源照亮它。我们为场景中的每个对象分别执行此操作。虽然很容易理解和实现，但它对性能的影响也相当大，因为每个渲染对象都必须为它的每个渲染片段迭代每个光源，这是非常多的！在深度复杂性高（多个对象覆盖同一屏幕像素）的场景中，正向渲染也往往会浪费大量可用的片段着色器资源，因为片段着色器输出会被覆盖。

Deferred shading or deferred rendering aims to overcome these issues by drastically changing the way we render objects. This gives us several new options to significantly optimize scenes with large numbers of lights, allowing us to render hundreds (or even thousands) of lights with an acceptable framerate. The following image is a scene with 1847 point lights rendered with deferred shading (image courtesy of Hannes Nevalainen); something that wouldn't be possible with forward rendering.

延迟着色或延迟渲染旨在通过彻底改变我们渲染对象的方式来克服这些问题。这为我们提供了几个新选项来显著优化具有大量光源的场景，使我们能够以可接受的帧速率渲染数百个（甚至数千个）光源。下图是使用延迟着色渲染 1847 个点光源的场景（图片由 Hannes Nevalainen 提供）；这是前向渲染无法实现的。

<p align="center">
  <img src="../../../../images/LearnOpenGL-AdvancedLighting-DeferredShading-DeferredExample.png">
</p>

Deferred shading is based on the idea that we defer or postpone most of the heavy rendering (like lighting) to a later stage. Deferred shading consists of two passes: in the first pass, called the geometry pass, we render the scene once and retrieve all kinds of geometrical information from the objects that we store in a collection of textures called the G-buffer; think of position vectors, color vectors, normal vectors, and/or specular values. The geometric information of a scene stored in the G-buffer is then later used for (more complex) lighting calculations. Below is the content of a G-buffer of a single frame:

延迟着色是基于这样一种想法，即我们将大多数繁重的渲染（如光照）延迟或推迟到稍后的阶段中。延迟着色由两个通道组成：在第一个通道（称为几何通道）中，我们渲染一次场景，并获取对象的各种几何信息（例如位置向量、颜色向量、法线向量和/或镜面反射值），将其存储在称为 G-buffer 的纹理集合中。然后，存储在 G-buffer 中的场景的几何信息稍后用于（更复杂的）光照计算。以下是单个帧的 G-buffer 的内容：

<p align="center">
  <img src="../../../../images/LearnOpenGL-AdvancedLighting-DeferredShading-Gbuffer.png">
</p>

We use the textures from the G-buffer in a second pass called the lighting pass where we render a screen-filled quad and calculate the scene's lighting for each fragment using the geometrical information stored in the G-buffer; pixel by pixel we iterate over the G-buffer. Instead of taking each object all the way from the vertex shader to the fragment shader, we decouple its advanced fragment processes to a later stage. The lighting calculations are exactly the same, but this time we take all required input variables from the corresponding G-buffer textures, instead of the vertex shader (plus some uniform variables).

我们在称为光照通道的第二通道中使用 G-buffer 的纹理，在该通道中，我们渲染一个填满屏幕的四边形，并使用存储在 G-buffer 中的几何信息计算每个片段的场景光照（我们逐个像素地遍历 G-buffer）。我们没有将每个对象从顶点着色器一路带到片段着色器，而是将其片段的高级处理解耦到后期阶段进行。光照计算完全相同，但这次我们从相应的 G-buffer 纹理中获取所有必需的输入变量，而不是从顶点着色器（和一些 uniform 变量）中。

The image below nicely illustrates the process of deferred shading.

下图很好地说明了延迟着色的过程。

<p align="center">
  <img src="../../../../images/LearnOpenGL-AdvancedLighting-DeferredShading-Overview.png">
</p>

A major advantage of this approach is that whatever fragment ends up in the G-buffer is the actual fragment information that ends up as a screen pixel. The depth test already concluded this fragment to be the last and top-most fragment. This ensures that for each pixel we process in the lighting pass, we only calculate lighting once. Furthermore, deferred rendering opens up the possibility for further optimizations that allow us to render a much larger amount of light sources compared to forward rendering.

这种方法的一个主要优点是，无论 G-buffer 中最终记录了什么片段，最终都会成为屏幕像素的实际片段信息。经过深度测试后，这个片段就是最后一个也是最前面的片段。这确保了在光照通道中，对于我们处理的每个像素，只会被计算一次光照。此外，延迟渲染为进一步优化提供了可能性，与前向渲染相比，我们可以渲染更多的光源。

It also comes with some disadvantages though as the G-buffer requires us to store a relatively large amount of scene data in its texture color buffers. This eats memory, especially since scene data like position vectors require a high precision. Another disadvantage is that it doesn't support blending (as we only have information of the top-most fragment) and MSAA no longer works. There are several workarounds for this that we'll get to at the end of the chapter.

不过，它也有一些缺点，因为 G-buffer 要求我们在其纹理颜色缓冲区中存储相对大量的场景数据。这会占用一些内存，尤其是类似位置向量之类的需要高精度的场景数据。另一个缺点是它不支持混合（因为我们只有最前面的片段的信息），并且 MSAA 不再有效。对于这些问题有几种解决方法，我们将在本章末尾介绍。

Filling the G-buffer (in the geometry pass) isn't too expensive as we directly store object information like position, color, or normals into a framebuffer with a small or zero amount of processing. By using multiple render targets (MRT) we can even do all of this in a single render pass.

填充 G-buffer（在几何通道中）成本不高，因为我们能以很少或为零的处理量，就可以直接将位置、颜色或法线等对象信息存储到帧缓冲区中。通过使用多渲染目标（MRT）技术，我们甚至可以在单个渲染过程中完成所有这些操作。

# The G-buffer

The G-buffer is the collective term of all textures used to store lighting-relevant data for the final lighting pass. Let's take this moment to briefly review all the data we need to light a fragment with forward rendering:

G-buffer 是为最终光照通道存储相关光照数据的所有纹理的统称。让我们借此机会简要回顾一下使用前向渲染照亮一个片段所需的所有数据：

* A 3D world-space position vector to calculate the (interpolated) fragment position variable used for lightDir and viewDir.
* An RGB diffuse color vector also known as albedo.
* A 3D normal vector for determining a surface's slope.
* A specular intensity float.
* All light source position and color vectors.
* The player or viewer's position vector.

With these (per-fragment) variables at our disposal we are able to calculate the (Blinn-)Phong lighting we're accustomed to. The light source positions and colors, and the player's view position, can be configured using uniform variables, but the other variables are all fragment specific. If we can somehow pass the exact same data to the final deferred lighting pass we can calculate the same lighting effects, even though we're rendering fragments of a 2D quad.

有了这些（每个片段）变量可以使用，就能够计算出我们习惯的（Blinn-）Phong 光照。这些变量里光源位置和颜色，以及玩家的视角位置，都可以使用 uniform 变量进行配置，但其他的都是特定于片段的。如果我们能以某种方式将完全相同的数据传递到最终的延迟光照通道，即使我们正在渲染 2D 四边形的片段，我们也可以计算出相同的光照效果。

There is no limit in OpenGL to what we can store in a texture so it makes sense to store all per-fragment data in one or multiple screen-filled textures of the G-buffer and use these later in the lighting pass. As the G-buffer textures will have the same size as the lighting pass's 2D quad, we get the exact same fragment data we'd had in a forward rendering setting, but this time in the lighting pass; there is a one on one mapping.

在 OpenGL 中，我们可以在纹理中存储的内容没有限制，因此将每个片段的所有数据储存在 G-buffer 的一个或多个填充的屏幕纹理中，并在稍后的光照通道中使用这些数据是可行的。由于 G-buffer 纹理的大小将与光照通道的 2D 四边形相同，因此在光照通道中，我们就获得了与前向渲染设置完全相同的片段数据;有一个一对一的映射。

In pseudocode the entire process will look a bit like this:

在伪代码中，整个过程看起来是这样：

```c++
while(...) // render loop
{
    // 1. geometry pass: render all geometric/color data to g-buffer 
    glBindFramebuffer(GL_FRAMEBUFFER, gBuffer);
    glClearColor(0.0, 0.0, 0.0, 1.0); // keep it black so it doesn't leak into g-buffer
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    gBufferShader.use();
    for(Object obj : Objects)
    {
        ConfigureShaderTransformsAndUniforms();
        obj.Draw();
    }  

    // 2. lighting pass: use g-buffer to calculate the scene's lighting
    glBindFramebuffer(GL_FRAMEBUFFER, 0);
    lightingPassShader.use();
    BindAllGBufferTextures();
    SetLightingUniforms();
    RenderQuad();
}
```

The data we'll need to store of each fragment is a position vector, a normal vector, a color vector, and a specular intensity value. In the geometry pass we need to render all objects of the scene and store these data components in the G-buffer. We can again use multiple render targets to render to multiple color buffers in a single render pass; this was briefly discussed in the Bloom chapter.

我们需要存储每个片段的位置向量、法向向量、颜色向量和镜面反射强度值数据。在几何通道中，我们需要渲染场景中的所有对象，并将这些数据部件存储在 G-buffer 中。正如在 Bloom 章节简要讨论的那样，我们可以再次使用多渲染目标技术，在单个渲染通道中将对象的不同类型数据分别渲染到多个颜色缓冲区。

For the geometry pass we'll need to initialize a framebuffer object that we'll call gBuffer that has multiple color buffers attached and a single depth renderbuffer object. For the position and normal texture we'd preferably use a high-precision texture (16 or 32-bit float per component). For the albedo and specular values we'll be fine with the default texture precision (8-bit precision per component). Note that we use GL_RGBA16F over GL_RGB16F as GPUs generally prefer 4-component formats over 3-component formats due to byte alignment; some drivers may fail to complete the framebuffer otherwise.

对于几何通道，我们需要初始化一个称为 gBuffer 的帧缓冲对象，该对象附加了多个颜色缓冲区和一个深度渲染缓冲区对象。对于位置和法线纹理，我们最好使用高精度纹理（每分量 16 或 32 位浮点数）。对于反照率和镜面反射值，我们可以使用默认纹理精度（每分量 8 位精度）。请注意，因为 GPU 通常更喜欢 4 分量格式而不是 3 分量格式，由于字节对齐，我们使用了 GL_RGBA16F 而不是 GL_RGB16F；否则，某些驱动程序可能无法完成帧缓冲创建。

```c++
unsigned int gBuffer;
glGenFramebuffers(1, &gBuffer);
glBindFramebuffer(GL_FRAMEBUFFER, gBuffer);
unsigned int gPosition, gNormal, gColorSpec;
  
// - position color buffer
glGenTextures(1, &gPosition);
glBindTexture(GL_TEXTURE_2D, gPosition);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA16F, SCR_WIDTH, SCR_HEIGHT, 0, GL_RGBA, GL_FLOAT, NULL);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, gPosition, 0);
  
// - normal color buffer
glGenTextures(1, &gNormal);
glBindTexture(GL_TEXTURE_2D, gNormal);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA16F, SCR_WIDTH, SCR_HEIGHT, 0, GL_RGBA, GL_FLOAT, NULL);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT1, GL_TEXTURE_2D, gNormal, 0);
  
// - color + specular color buffer
glGenTextures(1, &gAlbedoSpec);
glBindTexture(GL_TEXTURE_2D, gAlbedoSpec);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, SCR_WIDTH, SCR_HEIGHT, 0, GL_RGBA, GL_UNSIGNED_BYTE, NULL);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT2, GL_TEXTURE_2D, gAlbedoSpec, 0);
  
// - tell OpenGL which color attachments we'll use (of this framebuffer) for rendering 
unsigned int attachments[3] = { GL_COLOR_ATTACHMENT0, GL_COLOR_ATTACHMENT1, GL_COLOR_ATTACHMENT2 };
glDrawBuffers(3, attachments);
  
// then also add render buffer object as depth buffer and check for completeness.
[...]
```

Since we use multiple render targets, we have to explicitly tell OpenGL which of the color buffers associated with GBuffer we'd like to render to with glDrawBuffers. Also interesting to note here is we combine the color and specular intensity data in a single RGBA texture; this saves us from having to declare an additional color buffer texture. As your deferred shading pipeline gets more complex and needs more data you'll quickly find new ways to combine data in individual textures.

由于我们使用多个渲染目标，因此我们必须使用 glDrawBuffers 明确告诉 OpenGL，我们希望渲染到与 G-buffer 关联的哪个颜色缓冲区。同样值得关注的是，我们将颜色和镜面反射强度数据组合在一个 RGBA 纹理中，这使我们不必声明额外的颜色缓冲区纹理。随着延迟着色管线变得越来越复杂，并且需要更多数据，你会很快找到新的方法将数据组合到单个纹理中。

Next we need to render into the G-buffer. Assuming each object has a diffuse, normal, and specular texture we'd use something like the following fragment shader to render into the G-buffer:

接下来，我们需要渲染到 G-buffer 中。假设每个对象都具有漫反射、法线和镜面反射纹理，我们将使用类似于以下片段着色器的东西来渲染到 G-buffer 中：

```glsl
#version 330 core
layout (location = 0) out vec3 gPosition;
layout (location = 1) out vec3 gNormal;
layout (location = 2) out vec4 gAlbedoSpec;

in vec2 TexCoords;
in vec3 FragPos;
in vec3 Normal;

uniform sampler2D texture_diffuse1;
uniform sampler2D texture_specular1;

void main()
{    
    // store the fragment position vector in the first gbuffer texture
    gPosition = FragPos;
    // also store the per-fragment normals into the gbuffer
    gNormal = normalize(Normal);
    // and the diffuse per-fragment color
    gAlbedoSpec.rgb = texture(texture_diffuse1, TexCoords).rgb;
    // store specular intensity in gAlbedoSpec's alpha component
    gAlbedoSpec.a = texture(texture_specular1, TexCoords).r;
}
```

As we use multiple render targets, the layout specifier tells OpenGL to which color buffer of the active framebuffer we render to. Note that we do not store the specular intensity into a single color buffer texture as we can store its single float value in the alpha component of one of the other color buffer textures.

当我们使用多个渲染目标时，布局说明符会告诉 OpenGL 我们将渲染到活动帧缓冲区的哪个颜色缓冲区。请注意，我们不会将镜面反射强度存储到单个颜色缓冲区纹理中，因为我们可以将其单个浮点值存储在其他颜色缓冲区纹理的 alpha 分量中。

> **Note**<br>
请记住，在光照计算中，保证所有变量在一个坐标空间当中至关重要。在这里，我们在世界空间中存储（并计算）所有变量。

If we'd now were to render a large collection of backpack objects into the gBuffer framebuffer and visualize its content by projecting each color buffer one by one onto a screen-filled quad we'd see something like this:

如果我们现在要将大量背包对象渲染到 gBuffer 帧缓冲区中，并通过将每个颜色缓冲区一个接一个地投影到一个填满屏幕的四边形上来可视化其内容，我们会看到如下内容：

<p align="center">
  <img src="../../../../images/LearnOpenGL-AdvancedLighting-DeferredShading-VisualizedGbuffer.png">
</p>

Try to visualize that the world-space position and normal vectors are indeed correct. For instance, the normal vectors pointing to the right would be more aligned to a red color, similarly for position vectors that point from the scene's origin to the right. As soon as you're satisfied with the content of the G-buffer it's time to move to the next step: the lighting pass.

上图中世界空间位置和法向量的可视化尝试确实是正确的。例如，指向右侧的法向量将更符合红色，从场景原点指向右侧的位置向量同样也是这样。一旦你对 G-buffer 的内容感到满意，就该进入下一步了：光照通道。