---
title: Unity——NGUI染色遮罩Sharer
date: 2017-02-14 10:57:00
tags: Unity
categories: 游戏前端
photos:
- http://img.blog.csdn.net/20170214103154480
---

>简述：
遮罩的意思是指对原图被遮住的部分进行一定的处理，这里使用最简单的处理就是染色，所以我们需要创建一个遮罩层，通常使用另一个图片来作为遮罩层，也就是mask图。

<!--more-->
---
###一、mask图的作用：
跟UITexture使用的图片尺寸大小必须一致，将原图中需要被遮住（染色）的部分的位置在mask图中对应位置用纯色（例如：绿色（三原色的一种））涂满，学过flash动画的应该知道遮罩动画的原理，mask图其实就是一个遮罩层。

---
###二、sharer的实现思路：
其实就是获取原图UITexture上的每个像素点，然后根据坐标位置去获取mask图UITexture对应位置上指定颜色值的有无，然后乘以透明度（.a属性），例如：mask图为绿色，这获取mask图像素点的.g属性值，加入不为0，则该像素点要染色，否则则不染色，具体该像素点要染成什么颜色可以通过变量传给shader。

---
###三、实例：
下面是一个染色shader的代码：
```cg
Shader "Micro/TransparentMasked" {
	Properties {
		_MainTex ("Base (RGB)", 2D) = "white" {}
		_MaskTex ("Base (RGB)", 2D) = "white" {}
		_MyColor ("MyColor", Color) = (0,0,1,1)
	}
	
	Category
	{	
		Tags {"Queue"="Transparent" "IgnoreProjector"="True"}
		Lighting Off
		ZWrite On
		Blend One Zero
		SubShader {
			Pass {
				CGPROGRAM
				#pragma vertex vert_img
				#pragma fragment frag
	
				#include "UnityCG.cginc"
	
				uniform sampler2D _MainTex;
				uniform sampler2D _MaskTex;
				float4 _MyColor;
	
				float4 frag(v2f_img i) : COLOR 
				{
					float4 layer = tex2D(_MainTex, i.uv);					
					float4 mask = tex2D(_MaskTex, i.uv);
									
					float4 returnColor = layer;

					float gValue = max(0 ,mask.g - mask.b) * mask.a;

					returnColor = (returnColor * (1 - gValue)) + (_MyColor * returnColor) * gValue;

					returnColor.a = layer.a;
					return returnColor;
				}
				ENDCG
			}
		}
	}
}
```
####1.输入声明：
首先是输入的声明，这里我们需要明确进行此操作所需的输入有：原图（Texture）、遮罩图（Texture）、目标染色色值（float4），即可声明如下：
```
				uniform sampler2D _MainTex;
				uniform sampler2D _MaskTex;
				float4 _MyColor;
```

####2.处理函数：
关键部分在于每个像素点处理函数部分代码：
```
float4 frag(v2f_img i) : COLOR 
				{
					float4 layer = tex2D(_MainTex, i.uv);					
					float4 mask = tex2D(_MaskTex, i.uv);
									
					float4 returnColor = layer;

					float gValue = max(0 ,mask.g - mask.b) * mask.a;

					returnColor = (returnColor * (1 - gValue)) + (_MyColor * returnColor) * gValue;

					returnColor.a = layer.a;
					return returnColor;
				}
```
####3.源码解析：
接下来我们进行逐句解析：
- 首先，我们需要将两张图片传入到shader中，用两个float4四元矩阵layer和mask来保存图片数据，每个像素点的数据都是一个float4值，即表示（r,b,g,a）——>（红，绿，蓝，透明度），其中layer是原图层，mask为遮罩层；
- 通过``float gValue = max(0 ,mask.g - mask.b) * mask.a;``获得该像素点是否染色的状态；
- 计算每个像素点染色之后的结果``returnColor = (returnColor * (1 - gValue)) + (_MyColor * returnColor) * gValue;``，计算得到每个像素点的输出值；
- 透明度与原图保持一致``returnColor.a = layer.a;``。

---
###四、Unity实践：
- 1.新建一个Shader文件，步骤：在Project窗口中右键->Create->Shader，内容为上边的源码，命名为``MaskColor.shader``：
![MaskColor](http://img.blog.csdn.net/20170214103445766)
- 2.导入两张图片，原图和mask图：
![原图和mask图](http://img.blog.csdn.net/20170214103154480)
- 3.新建一个材质球，步骤：在Project窗口中右键->Create->Material，取名为选中``MaskColor.mat``，选中次材质球，将其Shader修改为我们刚刚创建的``MaskColor.shader``：
![MaskColor.mat](http://img.blog.csdn.net/20170214103401577)
假如一切正常，更改后选中该材质球，在Inspecteor窗口查看其属性可以看到：
![属性](http://img.blog.csdn.net/20170214103809974)
- 4.将Avatar.png拖入到材质球的第一个Texture中，将Avatar_mask.png拖入到材质球的第二个Texture中，修改MyColor的色值，这个色值就是我们的遮罩目标染色值，可以在预览窗口看到染色结果：
![](http://img.blog.csdn.net/20170214104456655)
- 5.为了更加直观地观察到染色结果，可以使用NGUI在场景中创建一个UITexture，将``MaskColor.mat``拖到其Material属性中，即可在Game视窗中看到结果：
![](http://img.blog.csdn.net/20170214105647261)
- 6.加入要动态修改染色的颜色，通过代码对MyColor属性进行赋值：
    ```C#
    using UnityEngine;
    using System.Collections;

    public class TestColor : MonoBehaviour {
        public UITexture texture;
        // Use this for initialization
        void Start () {
            //染成紫色
            texture.material.SetColor ("_MyColor", new Color(0.619f, 0.431f, 0.717f));
        }
    }
    ```
    得到新的染色结果:
    ![](http://img.blog.csdn.net/20170214110324296)

	最终资源文件结构：
	![](http://img.blog.csdn.net/20170214111019580)

####补充：
其实，Shader中的图片也是可以通过代码动态替换的，将散图放在``Resources``文件夹下，然后通过``Resources.Load(资源路径);``进行动态获取，然后通过材质的方法``material.SetTexture``进行赋值。