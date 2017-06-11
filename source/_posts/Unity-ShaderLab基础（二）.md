---
title: Unity ShaderLab基础（二）Cg语言
date: 2016-09-06 11:55:59
tags: Unity,ShaderLab,Cg
categories: Unity
---
学习一门新的语言，有几个基本需要掌握的：数据类型（基本数据类型和结构类型），基本语法（表达式和控制语句等），编译和运行方式，这一点在CPU编程语言和GPU编程语言中是相似的，Cg作为一门GPU图形学语言也是如此。
<!--more-->

## Cg的数据类型：
### 1.基本数据类型
Cg支持7种基本的数据类型，分别是：

- **float**， 32 位浮点数据，一个符号位。浮点数据类型被所有的 profile 支持
- **half**，16 为浮点数据
- **int**，32 位整形数据，有些 profile 会将 int 类型作为 float 类型使用
- **fixed**，12 位定点数，被所有的 fragment profiles 所支持
- **bool**，布尔数据，通常用于 if 和条件操作符（ ?: ） ，布尔数据类型被所有的profiles 支持
- **simpler***， 纹理对象的句柄（ the handle to a texture object ） ，分为 6 类：
sampler, sampler1D, sampler2D, sampler3D, samplerCUBE, 和 samplerRECT 。DirectX profiles 不支持 samplerRECT 类型， 除此之外这些类型被所有的 pixelprofiles 和 NV40 vertex program profile 所支持（ CgUsersManual 30 页） 。由此可见，在不远的未来，顶点程序也将广泛支持纹理操作
- **string**，字符类型，该类型不被当前存在的 profile 所支持，实际上也没有必要在 Cg 程序中用到字符类型，但是你可以通过 Cg runtime API 声明该类型变量，并赋值；因此，该类型变量可以保存 Cg 文件的信息。

>前6种类型为常用类型，**string**类型几乎不使用。

### 2.其他内置数据类型：

- **向量**

Cg还提供了内置的向量数据类型 (built-in vector data types) ，内置的向量数据类型基于基础数据类型。 例如： float4， 表示 float 类型的 4 元向量； bool4， 表示 bool类型 4 元向量。
（注意： **向量最长不能超过 4 元**， 即在 Cg 程序中可以声明 float1 、 float2 、 float3 、float4 类型的数组变量，但是不能声明超过 4 元的向量。）
向量初始化方式一般为：

	float4 array = float4(1.0, 2.0, 3.0, 4.0);
较长的向量还可以通过较短的向量进行构建：

	float2 a = float2(1.0, 1.0);  
	float4 b = float4(a, 0.0, 0.0);  

- **矩阵**

Cg还提供矩阵数据类型，不过**最大的维数不能超过4X4阶**，例如：

	float1x1 matrix1;//等价于float matirx1; x是字符，并不是乘号！  
	float2x3 matrix2;//表示 2*3 阶矩阵，包含6个float类型数据  
	float4x2 matrix3;//表示 4*2 阶矩阵，包含8个float类型数据  
	float4x4 matrix4;//表示 4*4 阶矩阵，这是最大的维数  
矩阵初始化：

	float2x3 matrix5 = {1.0, 2.0, 3.0, 4.0, 5.0, 6.0};  

- **数组**

**数组数据类型在Cg中的作用：作为函数的形参，用于大量数据的传递**，例如：顶点参数数组、光照参数数据等。

一维数组：

	float a[10];//声明了一个数组，包含 10 个 float 类型数据  
	float a[4] = {1.0, 2.0, 3.0, 4.0}; //初始化一个数组  
	int length = a.length;//获取数组长度  
多维数组：

	float b[2][3] = {{0.0, 0.0, 0.0},{1.0, 1.0, 1.0}};  
	int length1 = b.length; // length1 值为 2  
	int length2 = b[0].length; // length2 值为 3  


注：在Cg中，向量、矩阵与数组是完全不同的，向量和矩阵是内置的数据类型，而数组则是一种数据结构。

- **类型转换**

Cg 中的类型转换和 C 语言中的类型转换很类似。 C 语言中类型转换可以是**强制类型转换**，也可以是**隐式转换**，如果是后者，则数据类型从低精度向高精度转换。在 Cg 语言中也是如此：

	float a = 1.0;  
	half b = 2.0;  
	float c = a+b; //等价于 float c = a + (float)b;  
当有类型变量和无类型常量数据进行运算时，该常量数据不做类型转换，例如：

	float a = 1.0;  
	float b = a + 2.0;//2.0为无类型常量数据，编译时作为float 类型
Cg 语言中对于常量数据可以加上类型后缀，表示该数据的类型，例如：

	float a = 1.0;  
	float b = a + 2.0h;//2.0h为half类型常量数据，运算是需要做类型转换
常量的类型后缀有3种：

- f：表示float
- h：表示half
- x：表示fixed

---

## Cg的语法：
Cg的关系操作符、逻辑操作符、位移操作符都与C语言有相似之处，需要特别注意的是**Swizzle操作符**，例如：

	float4(a, b, c, d).xyz 	//等价于 float3(a, b, c)
	float4(a, b, c, d).xyy 	//等价于 float3(a, b, b)
	float4(a, b, c, d).wzyx //等价于 float4(d, c, b, a)
	float4(a, b, c, d).w 	//等价于 float d

---

## Cg的编译：
### 1.编译方式：
- **编译程序**：

计算机只能理解和执行由 0 、 1 序列（电压序列）构成的机器语言，所以汇编语言和高级语言程序都需要进行翻译才能被计算机所理解， 担负这一任务的程序称为语言处理程序，通常也被称为编译程序。

- **静态编译**：

一旦编译后，除非改变程序代码，否则不需要重新编译，这种方式称为静态编译（ static compilation ） 。静态编译最重要的特征是：**一旦编译为可执行文件，在可执行文件运行期间不再需要源码信息**。

- **动态编译**：

编译程序和源码都要参与到程序的运行过程中，就像脚本语言（Lua、JavaScrpit等），源码嵌套到调用的宿主语言程序中，运行时进行编译。

**Cg通常采用动态编译的方式（Cg也支持静态编译方式），即在宿主程序运行时，利用Cg运行库（Cg Runtimer Library）动态编译Cg代码。使用动态编译的方式，可以将Cg程序当做一个脚本，随时修改随时运行，节省时间，在OGRE图形引擎中就采用了这种方式。**

### 2.编译器：
Cg 编译器首先将 Cg 程序翻译成可被图形 API （ OpenGL 和 Direct3D ）所接受的形式， 然后应用程序使用适当的 OpenGL 和 Direct3D 命令将翻译后的 Cg 程序传递给图形处理器， OpenGL 和 Direct3D 驱动程序最后把它翻译成图形处理器所需要的硬件可执行格式。**NVIDIA 提供的 Cg 编译器为 cgc.exe**。

* 下载[Cg Toolkit](https://developer.nvidia.com/cg-toolkit-download)；
* 安装之后，在安装目录的Cg\bin中就有cgc.exe；
* 打开命令行窗口，输入 ```cgc -h``` ，假如不报错则说明安装成功。

### 3.Cg指令：

**编译指令**

	cgc [options] file

* ```[options]``` 表示可选配置项;
* ```file``` 表示 Cg 程序文件名。

例如，比较典型的编译方式：

	cgc -profile glslv -entry main_v test.cg
 
* ```-profile``` 是profile配置项名；
* ```glslv``` 是当前所使用的profile名称；
* ```-entry``` 着色程序的入口函数名称配置项；
* ```main_v``` 是顶点着色程序的入口函数名；
* ```test.cg``` 是当前的着色程序文件名（必须带后缀名）,Cg源码文件需以**.cg**为后缀名；

将Cg语言所写的着色程序转换为使用**GLSL**或**HLSL**所编写的程序：

	cgc –profile glslv –o direct.glsl –entry main_v test.cg
表示编译文件 test.cg 中的顶点着色程序， 入口函数名为 main_v ， 并将顶点着色程序转换为 glsl 程序，然后保存成文件 direct.glsl 。

**备注**：GPU编程，是无法跟踪调试着色程序的，一个着色程序，语法错误可以通过编译器发现，但是代码的逻辑错误只能认真查找。


>关于Cg的更详细的介绍可以参考这篇博客，利用OpenGL、C++和Cg进行Cg的测试：[【GPU编程】开始Cg之旅，编译自己的第一个Cg程序](http://blog.csdn.net/xiajun07061225/article/details/6937272)

### 4.**Cg Profiles:**
Cg 程序的编译不但依赖于宿主程序所使用的三维编程接口，而且依赖于图形硬件环境，因为图形硬件自身的限制，不一定支持某种 Cg 语句。
**被特定的图形硬件环境或 AIP 所支持的 Cg 语言子集，被称为Cg Profiles** 。

profile分为：**顶点程序的profile**和**片段程序的profile**，所以编译顶点着色程序时必须选用当前图形硬件支持的顶点profile ，同理，编译片段着色程序时必须选用当前图形硬件支持的片段profile 。

顶点 profile 和片段 profile 又基于 OpenGL 和 DirectX 的不同版本或扩展，划分为各种版本，当前 Cg compiler 所支持的 profiles 有：

	OpenGL ARB vertex programs
	        Runtime profile: CG_PROFILE_ARBVP1
	        Compiler option: _profile arbvp1
	OpenGL ARB fragment programs
	        Runtime profile: CG_PROFILE_ARBFP1
	        Compiler option: _profile arbfp1
	......


