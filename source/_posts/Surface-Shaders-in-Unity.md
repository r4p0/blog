---
title: Surface Shaders in Unity
date: 2017-11-29 15:09:44
tags: Unity Shader
---
Unity中Shader表面着色器基础
=======

[Unity云中客](http://www.jianshu.com/u/5e73de0977ca)

> 2016.04.17 22:01  
> 转载：[http://www.jianshu.com/p/b17503c056de](http://www.jianshu.com/p/b17503c056de)

写与灯光交互的Shader是很复杂的，因为有不同的灯光类型(light Type)，不同的阴影选项(Shadow Options)，不同的渲染路径(renderingPath：forward,deferred rendering)，所以Shader应该以某种方式来处理所有的复杂情况。

Unity中的==Surface Shaders==是一种比使用底层的顶点/像素着色器程序（vertex/pixel shader programs）写起来更为简单通用的方式。 surfaceShaders 也是用Cg/hlsl来写。

<!--more-->

## How It Works ##

首先你需要定义一个"Surface"函数来获取任何uv信息或者你需要的数据作为输入，最后会填充在输出结构体"SurfaceOutput"里，SurfaceOutput是对表面属性的基本描述(例如Albedo,Normal,Emission,Specular等等)，你可以将这些代码写在cg/hlsl里。
然后Surface Shaders计算编译需要哪些输入数据，以及哪些输出被填充等等，然后生成原生的vertex/pixed shaders，同时渲染通道会去处理forward和deferred渲染。
标准的表面着色器输出结构：

```hljs
struct SurfaceOutput
{
    fixed3 Albedo;  // diffuse color /反射颜色
    fixed3 Normal;  // tangent space normal, if written（法向量）
    fixed3 Emission;//自发光颜色
    half Specular;  // specular power in 0..1 range//镜面反射度
    fixed Gloss;    // specular intensity（光泽度）
    fixed Alpha;    // alpha for transparencies（透明度）
};
```

在Unity5中，SurfaceShader也可以使用光照模型的基本物理属性。内置的"**Standard**"和"**StandardSpecular**"光照模型分别使用下面的输出结构体：

（Standard和StandardSpecular光照模型对应普通没有物理的的Lambert何BlinnPhong光照模型）


```hljs
struct SurfaceOutputStandard
{
    fixed3 Albedo;      // base (diffuse or specular) color
    fixed3 Normal;      // tangent space normal, if written
    half3 Emission;
    half Metallic;      // 0=non-metal, 1=metal，金属度
    half Smoothness;    // 0=rough, 1=smooth，光滑度
    half Occlusion;     // occlusion (default 1)，遮挡率
    fixed Alpha;        // alpha for transparencies
};
```

```hljs
struct SurfaceOutputStandardSpecular
{
    fixed3 Albedo;      // diffuse color
    fixed3 Specular;    // specular color
    fixed3 Normal;      // tangent space normal, if written
    half3 Emission;
    half Smoothness;    // 0=rough, 1=smooth
    half Occlusion;     // occlusion (default 1)
    fixed Alpha;        // alpha for transparencies
};
```

对比SurfaceOutput和SurfaceOutputStandard，会发现前者的结构中Specular和Gloss被替换成在后者的结构中的Metallic，Smoothness，Occlusion三个属性。这三个属性就是Unity5里新加的物理属性。同样比较SurfaceOutputStandard和SurfaceOutputStanaedSpecular结构体，发现Metallic属性被Specular属性替换掉。

所以Unity5.x基于光照模型添加的基本物理属性应该是在Specular镜面发射上做了一些处理，添加了遮挡和粗糙度的属性以及混合Specular而形成的Metallic属性，使物体表面具有更加接近真实的物理光泽效果，从而减轻开发者自己书写相似效果的工作量。

**##Surface Shader compile directives**（表面着色器编译指令）**

表面着色器和其它Shader一样被放在CGPROGRAM……ENDCG块中执行，不同的是：


- 代码必须写在子着色器块（SubShader）而不是通道（Pass）中，表面着色器会自己把它编译到多个通道里。
- 代码必须使用#pragma surface ... 指令去指示它的表面着色器。


“#Pragma surface” 指令是：

```#pragma surface surfaceFunction lightModel [optionalparams]```

### 必要参数 ###

- surfaceFunction - 含有表面着色器代码的cg函数。这个函数必须是```void surf(Input IN,inout SurfaceOutput o)```的形式，Input是一个自己定义的结构体，Input结构体里应该包含一些纹理坐标和其它的surface函数需要的其它的可以被自动识别的变量（autonmatic variables）。
- lightmodel - 使用的光照模型。内置一些基于物理的```Standard```和```StandardSpecular```光照模型，以及一些没有基于物理的Lambert（Diffuse）和BlinnPhong（specular）光照模型，当然也可以自己写光照模型（参见：[自定义光照模型](_blank)）
- Standard光照模型使用SurfaceOutputStandard作为输出结构体，对应Unity中的Standard（Metallic workflow）Shdaer。
- StandardSpecular光照模型使用SurfaceOutputStandardSpecular作为输出结构体，对应Unity中的Standard（Specular workflow）Shader
- Lambert和BlinnPhong光照模型是不基于物理的（大部分在Unity4.x中）,但是在低端设备上使用会更加快，性能更好。


### **可选参数** ###

**Transparency和Alpha testing**是由alpha和alphatest指令控制。不透明度主要分为两种：传统的alpha混合（一般用于淡出物体）或更加看似物理的“premultiplied blending”（允许半透明表面保留适当的镜面反射效果）。使半透明能够使用生成的表面着色器代码包含blending指令，而能够让alpha基于已有的变量剔除被生成的pixed shader丢弃的片段。

- **alpha** 或者 **alpha:auto**： 将会为简单的光照函数挑出淡出透明度（就像alpha:fade），并且为基于物理的光照函数预乘透明度（就像 alpha:premul）。
- **alpha:fade**：使用传统的淡出透明。
- **alpha:premul**：使用预乘alpha透明度。
- **alphatest : variableName**：使用alpha裁剪透明度。剔除值是一个浮点型变量variableName，你可能也会想使用addshadow指令去生成适当的阴影接受通道（shadow caster pass）。
- **keepalpha**：默认不透明（Opaque）选项，不管输出结构的Alpha或者光照函数的返回值都将表面着色器将Alpha通道值设置为1.0。使用这个选项允许保持光照函数的返回alpha值，即使使用的是不透明表面着色器。
- **decal:add**： 额外的贴花着色器（例如，terrian AddPass），这意味着物体在其它表面之上使用additive blending。
- **decal:blend**：半透明贴花着色器，这意味物体可以在其它物体表面使用alpha blending。


**自定义函数**可以被用来改变或计算来自顶点的数据，或者改变最后计算得出的片段颜色。


- **vertex:VertexFunction**,用户自定义的顶点修改函数。这个函数会在生成的顶点着色器开始被调用，并且可以修改或计算每个顶点的数据。
- **finalcolor:ColorFunction**，自定义的最终颜色修改函数。

**Shadows and Tessellation**（阴影和镶嵌花纹），可以控制如何处理阴影和镶嵌花纹的额外指令。

- **addshadow**,生成一个阴影投射通道，通常被用在自定义顶点修改函数中，因为阴影投射也获得程序顶点动画。通常shaders不需要任何特殊的shadows处理，因为可以使用它们FallBack里的阴影投射通道。
- **fullforwardshadows**,支持正向渲染路径（forward render path）的所有的灯光阴影。默认的着色器仅支持来自directional light的正向渲染（为了节省内部着色器变量数量），如果需要point light和spotlight在正向渲染中的阴影，可以直接使用这个。
- **tessellate:TessFunction**,使用DX11 GPU Tessellation，函数计算镶嵌纹理因子。


**Code Generation**（代码生成） 选项，通过默认生成的表面着色器代码试图去处理各种可能的lighting/shadowing/lightmap情况，然而有些情况下你不需要他们中的一些内容，这里你就可以调整代码生成去跳过它们。这样的结果会使你的shader更小并且被更快的加载。


- **exclude_path:deferred,forward,prepass**.不要为已经给的渲染路径生成pass，
- **noshadow**，不支持阴影接收。
- **noambient**，不适用于环境光或者light probes。
- **novertexlights**,不适用于正向渲染（forward rendering）的任何light probes或者逐顶点光照。
- **nolightmap**,不支持光照贴图。
- **nodynlightmap**,不支持动态的全局光照。
- **nodirlightmap** , shader不支持定向光照贴图。
- **nofog**,不支持所有的内置雾效。
- **nometa**,不生产”meta“通道，常被用于lightmapping和动态全局照明去提取表面信息。
- **noforwardadd**，不适用正向渲染额外的通道。这个使shaders支持一个全局光，而使用其它灯光逐顶点计算。可以使shader更小更有效率。


**其它**

- **softvegetation**,使表面着色器仅仅在软植被上的时候被渲染。
- **interpolateview**，计算在顶点着色器上视觉方向并且进行差值来代替在像素着色器中计算。可以使像素着色器更快，但是会使用多张贴图进行差值。
- **halfasview**,通过half-direction向量进入光照函数来代替视角方向，half-direction会被逐顶点计算并归一化。这会更快，但并不总是正确。
- **approxview**，Unity5中移除了，使用interpolateview代替。
- **dualforward**,在正向渲染路径中使用贴花光照贴图。

### **表面着色器输入结构** ###

在输入结构体```Input```中通常会有一些shader使用需要的纹理坐标。纹理坐标必须以```uv```命名开头，后面街上纹理名称（或者以uv2开头，使用第二个纹理坐标）。

其它可以被放到Input结构中的值有：

- **float3 viewDir**,将含有视角方向，用于计算视差效果，边缘光照等。
- **float4**加上**COLOR**语意绑定，将会包含差值的逐顶点颜色。

- float4 screenPos,将包含屏幕空间位置，用于反射后屏幕空间效果。
- **float3 worldPos**,包含世界空间位置信息。
- **float3 worldRef1**,如果表面着色器没有写给o.Normal，将包含世界反射向量。
- **float3 worldNormal**,如果表面着色器没有写给o.Normal，将包含世界法向量。
- **float3 worldRef1;INTERNAL_DATA**,如果表面着色器没有写给o.Normal，将包含世界反射向量。使用WorldReflectionVector (IN, o.Normal)基于逐顶点法线贴图获取反射向量。
- **float3 worldNomal;INTERANT_DATA**，如果表面着色器没有写给o.Normal，使用WorldReflectionVector (IN, o.Normal)基于逐顶点法线贴图获取法线向量。

现在有部分表面着色器编译管线不支持DX11上特殊的HLSL语法，所以如果你使用HLSL一些特性，比如StructuredBuffers,RWTextures和其它的DX9没有的特性，可以使用宏包裹住。