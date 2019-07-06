---
title: Unity ShaderLab基础（三）Unity创建一个Shader
date: 2016-09-07 15:32:31
tags: 
  - Unity
  - ShaderLab
categories: Unity
---
Unity引擎是一个非常强大的支持跨平台开发的游戏引擎，基于Mono这个开源.Net的框架设计而成，在Unity中定义了**ShaderLab**来组织Shader的内容，针对不同平台进行编译。了解了Shader和Cg的一些基础知识之后，接下来我们要做的就是：学会如何在Unity中使用Cg编写Shader并实现一些简单的Shader效果。

<!--more-->

## Unity Shader：
说到底，Shader其实只是一段规定好输入（颜色，贴图等）和输出（渲染器能读懂的点和颜色的对应关系）的程序。那么，**设计一个Shader的过程其实就是根据输入，进行计算变换从而产生输出而已**。

### 1.分类：
在Unity中的Shader分为两类：

- **表面着色器**（Surface Shader）:已经为我们完成了大部分的工作，只需要简单的操作即可得到不错的效果；
- **片段着色器**（Fragment Shader）:可以自己设计出很多东西，因为可自行设置的内容更多，但也更加难写。使用片段着色器的目的是可以在更加底层进行更复杂（或者针对目标设备更高效）的开发。

### 2.Shader程序基本结构：
使用Unity中的框架来编写Shader程序，其实相对于其他游戏引擎要简单一些，在Cocos2d中还得从OpenGL层面开始编写逻辑，但是在Unity只需要往框架中填入需要控制的内容即可，一个Shader程序的基本结构如下图所示：

![](http://i.imgur.com/a6T9fF8.png)

- 首先，定义一些属性，用来指定代码将有哪些输入；
- 其次，会有一个或者多个子着色器，但是在实际运行中哪一个子着色器被使用是由运行的平台所决定的；
- 子着色器是代码的主体，每个子着色器包含一个或多个Pass；
- 最后指定一个回滚，用来处理所有Subshader都不能运行的情况（比如设备太老）。

执行着色时，平台选择最优先可以使用的子着色器，然后依次执行该子着色器中的Pass，然后输出结果。


### 3.Unity创建第一个Shader：
在Unity的Project面板中，``右键``-``Create``-``Shader``，取名为``Diffuse_Texture``，使用VS打开可以查看新建的Shader的内容如下所示：

```C#
	Shader "Custom/Diffuse_Texture" {
		Properties {
			_MainTex ("Base (RGB)", 2D) = "white" {}
		}
		SubShader {
			Tags { "RenderType"="Opaque" }
			LOD 200
			
			CGPROGRAM
			#pragma surface surf Lambert
	
			sampler2D _MainTex;
	
			struct Input {
				float2 uv_MainTex;
			};
	
			void surf (Input IN, inout SurfaceOutput o) {
				half4 c = tex2D (_MainTex, IN.uv_MainTex);
				o.Albedo = c.rgb;
				o.Alpha = c.a;
			}
			ENDCG
		} 
		FallBack "Diffuse"
	}
```

接下来我们要做的，就是解析这个Shader中每一行的含义和作用，包括了属性、Tags、LOD、光照模型等。

#### **解析：**
第一行指定了此Shader的名字，严格来说是指定了它的路径，在材质面板中选择Shader时，我们可以根据这个路径找到此Shader。

#### **属性**

在``Properties{}``块中定义的内容就是着色器的属性，【可以理解为一些CPU语言例如java类在开始处定义的一些属性（全局变量或常量）】，**这些属性将作为输入提供给所有的子着色器**。每个属性定义的语法：
```C#
	_Name("Display_Name",type) = defaultValue[{options}]
```
* ``_Name``：属性的名称，或者理解为变量名，在之后整个	Shader代码中通过此名称获取属性内容；
- ``Display_Name``：此字符串是Shader在Unity的材质编辑器中作为Shader可视化信息；
* ``type``：此属性的类型，Unity中支持的类型有：
   - Color：颜色，由RGBA(红绿蓝和透明度)四个量定义；
   - 2D：一张2的阶数大小（256，512等）的贴图，此贴图将在采样后被转为对应基于模型UV的每个像素的颜色，最终显示出来；
   - Rect：一个非2阶数大小的贴图；
   - Cube：即Cub map texture(立方体纹理)，即6张有联系的2D贴图的组合，主要用来做**反射效果**（比如：天空盒和动态反射），也会被转换为对应点的采样；
   - Range(min,max)：一个介于最大值max和最小值min之间的浮点数，一般用作调整Shader某些特性的参数（例如：透明度从0到1）；
   - Float：一个浮点数；
   - Vector：一个四维数；
* ``defaultValue``：定义的这个属性的默认值或者初始值，但不同属性类型的默认值格式不同，例如：
   - Color：咦0~1定义的rgba颜色，可以赋值(1,1,1,1);
   - 2D/Rect/Cube：贴图默认值需要是一个代表tini颜色的字符串，可以是空字符串或者"white","black","gray","bump"中的一个；
   - Float、Range：任意浮点数即可；
   - Vector：四维数，格式(x,y,z,w)；
* ``{option}``：只对2D、Rect和Cube贴图有关，初始值至少要在贴图后面写一对空白的``{}``，当需要打开特定选项时可以吧其写入到此花括号中，多个选项以空白分隔。可能的选项：ObjectLinear, EyeLinear, SphereMap, CubeReflect, CubeNormal，这些都是OpenGL中TexGen的模式。

**例子：**

```C#
	//颜色输入
	_MainColor ("Main Color", Color) = (0,0.5,1,0.5)
	//2的阶数大小的贴图输入
	_Texture ("Texture", 2D) = "white" {}
```

#### **Subshader**
上面已经解析了Shader代码的第一部分，接下来我们要将的就是Shader的代码主体，即SubShader的内容，在``SubShader{}``中的内容就是一个SubShader。

##### **(1) Tags**
SubShader中的第一句就是：
```C#
Tags { "RenderType"="Opaque" }
```
这是SubShader的标签，因为表面着色器可以被若干个标签（tags）所修饰，而**硬件正是通过判定这些标签（Tags）来决定什么时候调用该着色器**。所以，我们例子中的这一句``"RenderType"="Opaque"``的意思：告诉系统应该在渲染非透明物体时调用此SubShader，这与RenderType是Opaque是相对应的。

此外，**Tags其实也暗示了此Shader的输出情况**，例如：输出中都是半透明的物体，那就写在Opaque里；如果想渲染透明或者半透明的像素，那就应该写在Transparent里。

另外比较有用的标签还有：

- ``"IgnoreProjector"="True"``：不被Projects影响；
- ``"ForceNoShadowCasting"="True"``：从不产生阴影；
- ``"Queue"="xxx"``：**指定渲染顺序队列**。在Unity中，如果需要进行透明和不透明物体混合时，可能会遇到不透明物体无法呈现在透明物体之后的情况，这是由于Shader的渲染顺序不正确导致的。Queue指定物体渲染顺序，预定义的Queue有：
	- Background：最早被调用的渲染，用于渲染天空盒或者背景；
	- Geometry：默认值，用来渲染非透明的物体；
	- AlphaTest：用来渲染经过Alpha Test的像素，单独为AlphaTest设定一个Queue是出于对效率的考虑；
	- Transparent：以后从后往前的顺序渲染透明物体；
	- Overlay：用来渲染叠加的效果，是渲染的最后阶段（比如镜头光晕等特效）；

以上这些预定义的值，本质上是一组定义**整数**，Background = 1000， Geometry = 2000, AlphaTest = 2450， Transparent = 3000，最后Overlay = 4000。当然，在实际设置Queue值时，不仅可以使用上述的预定义值，还可以指定自己的Queue值，例如：``"Queue"="Transparent+100"``，表示一个在Transparent之后100的Queue上进行调用。

**通过调整Queue值，我们可以确保某些物体一定在另一个物体之前或之后被渲染**。

##### **(2) LOD**
第二行中的内容：

```C#
	LOD 200
```

LOD，即Level of Detail，这其实是**Unity的内建Diffuse着色器的设定值，决定了我们能够用什么样的Shader**。在Unity的Quality Settings中，我们可以设置允许的最大LOD，当设定的LOD小于SubShader的LOD时，这个SubShader将不可用。

Unity内建Shader定义了一组LOD的数值，我们在实现自己的Shader的时候可以将其作为参考来设定自己的LOD数值，这样在之后调整根据设备图形性能来调整画质时可以进行比较精确的控制。

- VertexLit及其系列 = 100
- Decal, Reflective VertexLit = 150
- Diffuse = 200
- Diffuse Detail, Reflective Bumped Unlit, Reflective Bumped VertexLit = 250
- Bumped, Specular = 300
- Bumped Specular = 400
- Parallax = 500
- Parallax Specular = 600

##### **(2) CGPROGRAM...ENDCG**
用``CGPROGRAM``开始和``ENDCG``结束，表明这部分是Cg代码。这是SubShader的主体部分，我们前面已经提到了属性中定义了此Shader的输入，那么此处代码的作用，便是对输入进行处理，并输出。接下来我们逐句进行解析：

- ``#pragma surface surf Lambert``：这是一个编译指令，声明此Shader是一个表面着色器，并指定了着色器的自动调用的函数名称为surf,而且指定光照模型为Lambert(普通的diffuse)，它的一般语法如下：

```CG
#pragma surface surfaceFunction lightModel [optionalparams]
```

   - surface：声明的是一个表面着色器
   - surfaceFunction：着色器代码的方法名称，着色器其作用时被调用
   - lightModel：使用的光照模型

---

- ``sampler2D _MainTex;``：其中``sampler2D``是GLSL中2D贴图的类型，类似的还有sampler1D、sampler3D、samplerCube等格式，主要用于存储texture数据。``_MainTex``是与之前在Proterties属性模块中声明的图贴所对应的，因为这个Shader是由两个独立的程序块组成的：外部的属性声明和回滚等Unity可以直接使用和编译的ShaderLab；而在``CGPROGRAM...ENDCG``中的代码块，是一段CG程序。**假如要在CG程序中访问Proterties中所定义的变量，必须使用和之前的变量相同的名字进行声明**。所以此句Cg代码的作用就是**再次声明并链接_MainTex，使接下来的Cg程序能够使用此变量**。
	
---
- Input结构体：这其实是用来把需要参与计算的数据封装起来，然后作为输入参数传入到下面surf函数中使用的，而且必须以``Input``命名。

```
struct Input {
	float2 uv_MainTex;
};
```
这里我们的Input结构体很简单，只是定义了一个float2类型的变量，这是Cg的数据类型，表示2个float类型的数据打包在一起，所以此处``uv_MainTex``表示的就是包含两个浮点数的变量，类似的还有float3和float4。

这里以uv作为前缀，其实UV mapping的作用是将一个2D贴图上的点按照一定规律映射到3D模型上，是3D渲染中最常见的一种顶点处理手段。所以在Cg中，在一个贴图变量（例如这里的_MainTex）前面加上uv，表示提取它的uv值（其实就是两个代表贴图上点的二维坐标）。后面的**surf函数中直接通过访问uv_MainTex来去的这张贴图当前需要计算的点的坐标值**。

---

- surf函数：这是在之前``#progma``中指定的着色器的调用方法，这也是着色器最核心的部分，这个方法的定义需要按照规定：**第一个参数是一个Input结构，第二个参数是一个inout的SurfaceOutput结构**。
```
void surf (Input IN, inout SurfaceOutput o) {
	half4 c = tex2D (_MainTex, IN.uv_MainTex);
	o.Albedo = c.rgb;
	o.Alpha = c.a;
}
```
上面已经说过了Input结构体的定义和使用，**在计算输出时，Shader会多次调用surf函数，每次给入一个贴图上的点坐标，用来计算输出**。

surf的第二个参数是一个可写的``SurfaceOutput``，SurfaceOutput是一个预定义的输出结构，我们的surf函数的目标就是根据输入把这个输出结构填上。SurfaceOutput结构体的定义如下：
```bash
struct SurfaceOutput {
    half3 Albedo;     //像素的颜色
    half3 Normal;     //像素的法向值
    half3 Emission;   //像素的发散颜色
    half Specular;    //像素的镜面高光
    half Gloss;       //像素的发光强度
    half Alpha;       //像素的透明度
};
```
half其实跟float与double类似，都是浮点数，只是精度不同，half称为半精度浮点数。

这个例子中，surf的代码如下：
```bash
	half4 c = tex2D (_MainTex, IN.uv_MainTex);
	o.Albedo = c.rgb;
	o.Alpha = c.a;
```
这里的``tex2D``函数是Cg中用来在一张贴图中对点进行采样的方法，返回一个float4。这里，我们队_MainTex在输入点上进行采样，并将其颜色的rgb值``c.rgb``赋给输出的像素颜色``o.Albedo``，将透明度``c.a``赋值给输出像素透明度``o.Alpha``。


#### 参考链接：

- 了解Shader的基础知识：[猫都能学会的Unity3D Shader入门指南（一）](https://onevcat.com/2013/07/shader-tutorial-1/)
- Unity官方关于Shader的一些资料：[Materials, Shaders & Textures](https://docs.unity3d.com/Manual/Shaders.html)
- 了解Shader的机制：[【Unity Shaders】初探Surface Shader背后的机制](http://blog.csdn.net/candycat1992/article/details/39994049)

#### 推荐书籍：

- 《Unity Shader and Effect Cookbook》，中文版：《Unity着色器和屏幕特效开发秘笈》
- 《GPU 编程与CG 语言之阳春白雪下里巴人》