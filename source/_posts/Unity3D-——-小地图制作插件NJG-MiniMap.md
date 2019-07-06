---
title: Unity3D —— 小地图制作插件NJG MiniMap
date: 2016-10-11 10:22:53
tags: 
  - NJG
  - Unity
categories: Unity
---

在很多实时PVP对战游戏（如：英雄联盟、王者荣耀等）的战斗场景中，都会有一个小地图，用于实时地显示一些比较重要因素，例如：队友和对手位置、存活炮塔位置、Boss出生死亡情况等。

### NJG MiniMap插件：

#### 1.下载地址：
NJG下载地址：链接：[http://pan.baidu.com/s/1kTkzxxt](http://pan.baidu.com/s/1kTkzxxt) 密码：jwqy

<!--more-->

#### 2、NGUI Version：
下载好插件后，导入到Unity中不用说，导入后可以看到NinjutsuGames文件夹，插件的所有内容都在这个文件夹下，找到NinjutsuGames/NJG MiniMap目录下的NGUI Version包双击，它会生成一个``NGUI Version``文件夹：

![](http://i.imgur.com/R6Glx1B.png)

#### 3、实例：
可以在NGUI Version/Examples/Scene2中查看示例，打开示例场景``Example-BigTerain``查看效果，但是好像会有一个BUG，查看世界地图的时候会出现显示错误，可看到图中红色箭头部分：

![](http://i.imgur.com/p4SuWBj.png)
![](http://i.imgur.com/vbAVmJm.png)

### 创建自己的Demo：

看过官方的示例，我们可以自己创建一个场景来试试，这里我就不搭建自己的场景，直接用NJG MiniMap搭建好的场景来做：

#### 1.新建场景：
新建一个场景，这里命名为scene9，找到``NinjutsuGames\NJG MiniMap\ExamplesAssets\Prefabs``目录下的``Scene.prefab``直接拖动到Hierarchy栏中，运行可以看到效果如下,这时候有些对象里可能会出现如下错误：

![](http://i.imgur.com/rjBtQFX.png)
![](http://i.imgur.com/R740xO4.png)

这是因为预设里面已经绑定了相关小地图的脚本，但是现在我们还没有添加相关小地图的NGUI内容，有两个解决方法：

- （1）直接删掉这个脚本；
- （2）待后续添加相关内容即可

这里把Scene里全部对象的这个脚本都删掉；

#### 2添加小地图:
小地图是用NGUI创建的

- 先用NGUI创建一个2D UI：NGUI——>Create——>2D UI：
- 把NinjutsuGames\NJG MiniMap\NGUI Version\Prefabs目录下的``NJG MiniMap.prefab``文件直接拖动到UI Root下面，点击UI Root下的Camera，可以在Scene的右下角看到小地图的缩略版：

这时候我们点击运行，可以看到小地图已经出现，截图如下：

![](http://i.imgur.com/hMgtWtm.png)

#### 3.添加其他物体：
小地图中没有任何标识，我们需要为小地图创建主角对象以及一些敌方怪物啊、NPC等等。在目录NinjutsuGames\NJG MiniMap\Common\Scripts\Core找到脚本``NJGMapItem.cs``，添加到要标识的对象上，在这个场景中我们以Scene中的`` _Player ``为例，将脚本添加到`` _Player ``组件中，然后选择NJGMap Item(Script)中的Market Type选项，这里我们选为Me，就可以在小地图中看到表示_Player对象的标识了：

![](http://i.imgur.com/ACoSWra.png)

#### 4.自定义图标：
我们还可以选择自定义图标，选择_Player的NJGMap Item组件中的``Edit NJG MiniMap``来进行编辑：

![](http://i.imgur.com/h97uTEG.png)

如下图：

![](http://i.imgur.com/fQ4KPzi.png)

- Altas选择自定义图标所在的图集；
- 点击Add New添加新的Market Type；
- Marker Type设置当前标识名；
- Icon Sprite就是选择对应的图标了；

#### 5.分层编辑：
按步骤3中修改对应的Marker Type的名字即可，效果如下：

![](http://i.imgur.com/uCcRIE2.png)

#### 6.额外功能：
在小地图中还可以添加迷雾效果：
选中UI Root下的NJG MiniMap，找到NJGMap组件中的FOW项，勾选上Enabled项：

![](http://i.imgur.com/nq9ZiH5.png)

这时候小地图已经被迷雾覆盖，还需要设置对象物体的可视，勾选_Player中NJGMap Item中的Reveal FOW选项，调节可视距离即可，可视距离为0的话默认全部可视：

![](http://i.imgur.com/H8CrEee.png)

#### 7、最终结果：

![](http://i.imgur.com/Cl9Ru6r.png)


