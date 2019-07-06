---
title: Unity3D —— protobuf导excel表格数据
date: 2016-09-18 20:21:47
tags: 
  - protobuf
  - Unity
  - excel
categories: Unity
---

### 前言：

之前使用``NPOI插件``编写的导表工具，其实就是直接将数据进行**序列化**，解析时还需要进行**反序列化**，步骤比较繁复，最近看到Google的一个开源的项目``protobuf``，不仅可以用于进行excel表格数据的导出，还能直接用于网络通信协议的定制。

### 一、protobuf简介：

protobuf是由google公司发布的一个开源的项目，是一款方便而又通用的数据传输协议。所以我们在Unity中也可以借助protobuf来进行数据存储和网络协议两方面的开发，这里先说说数据存储部分的操作，也就是：**将.xls表格数据通过protobuf进行序列化，并在Unity中使用。**

<!--more-->

#### 1.下载资源：
- [python2.7安装和配置](http://blog.csdn.net/linshuhe1/article/details/52056864)
- [protobuf-2.5.0.zip](http://pan.baidu.com/s/1kVk5LhP)
- [protobuf-net](https://github.com/mgravell/protobuf-net)

#### 2.流程图：
![](http://img.blog.csdn.net/20160818164221144)
从上图可看出基本的操作步骤：

- .xls表格文件，先通过``xls_deploy_tool.py``生成对应的.data文件和.proto文件，其中.data文件就是表格数据序列化后的结果，而.proto文件则是用于生成反序列化时使用的解析类的中间状态；
- 解析类.proto经过``protoc.exe``转换成.desc文件，用于后面通过protobuf-net等工具转化为特定的语言，这里我们需要得到的是C#解析类，即.cs类；
- 在Unity中导入``protobuf-net.dll``库，在C#代码中调用上述生成的.cs解析类来解析.data中的数据。

### 二、导表环境配置：
#### 1.Python相关配置：
由于从.xls文件生成.data和.proto，Python需要依赖``Proto库``和``xlrd库``，安装配置步骤：
        
- **setuptools**：这是Python的组件安装管理器，需要在安装protobuff组件前进行安装，到[setuptools官网](https://pypi.python.org/pypi/setuptools#downloads)下载插件的安装包，解压到指定目录，然后使用命令行进入安装包目录，执行指令：
```bash
python setup.py install；
```

- **Protobuff**：首先，我们将之前下载好的**源码包protobuf-2.5.0.zip**和**编译包protoc-2.5.0-win32.zip**压缩包解压到指定目录，路径最好不要包含中文；
   - 这里我解压protobuf-2.5.0.zip到的位置是“E:\Unity_Workplace\protobuf_250”；
   - 然后复制protoc-2.5.0-win32.zip解压得到的**protoc.exe**到protobuf_250\src目录下；
   - 在protobuf-2.5.0\python\google\protobuf下创建一个文件夹命名为**compiler**（安装完成后会在此目录下生成两个文件__init__.py和plugin_pb2.py）；
   - 使用命令行进入到解压后的目录下面的**Python目录**，执行：
```bash 
python setup.py install；
```

- **xlrd(xls reader)**：这其实是读取xls表格数据的一个工具插件，到[xlrd官网](https://pypi.python.org/pypi/xlrd)下载xrld的安装包，解压安装包然后使用命令行进入安装包目录，执行指令：
```bash 
python setup.py install。
```

#### 2.导表外部工具：
- **xls_deploy_tool.py**：这个工具其实是github上的一个开源的符合protobuff标准的根据excel自动生成匹配的PB的定义（``.proto文件``）并将数据序列化后生成二进制数据或者文本数据（``.data文件``）的一个工具，github下载地址：[xls_deploy_tool.py](https://github.com/jameyli/tnt/tree/master/python)。

- **protoc.exe和protogen.exe**：通过上面的工具，我们得到了两个文件：存储数据的.data文件和用于解析数据的.proto文件，但是我们在真正使用解析类来进行数据文件的解析时，必须是高级语言，当然protobuf-net提供很多种高级语言的支持。就像我们在Unity中我们使用的是C#语言，这需要两个工具来实现，一个是protobuf-2.5.0中的``protoc.exe``将.proto文件转换为“FileDescriptorSet”中间格式；另一个是使用protobuf-net中的``protogen.exe``，将中间格式的文件转换为最终状态，即高级语言的解析类.cs文件。

- 可以到github上下载protobuf-net的源码：[protobuf-net](https://github.com/linshuhe/protobuf-net)，下载后解压到本地，然后进入到解压后protobuf-net-master\protobuf-net目录下，通过Visual Studio打开protobuf-net.csproj：
![](http://img.blog.csdn.net/20160822105531605) ![](http://img.blog.csdn.net/20160822105539356)

- 编译完成后在当前目录下面的bin\Release目录下，生成了编译后的文件，其中我们需要的是protobuf-net.dll：
![](http://img.blog.csdn.net/20160822112150454)

- 将protobuf-net.dll复制到protobuf-net-master\ProtoGen目录下，用Visual Studio打开ProtoGen.csproj，参照上面步骤编译ProtoGen项目，得到protobuf-net-master\ProtoGen\bin\Release目录下面的protogen.exe及一些额外的文件，但在真正使用时此目录下面的所有文件都是必须的：
![](http://img.blog.csdn.net/20160822112506834)

### 三、样例：
#### 1.建立表格.xls：
当然使用此工具进行导表的表格需要符合指定的格式，根据xls_deploy_tool.py的备注内容：
```bash
# 说明:  
#   excel 的前四行用于结构定义, 其余则为数据，按第一行区分, 分别解释：  
#       required 必有属性  
#       optional 可选属性  
#           第二行: 属性类型  
#           第三行：属性名  
#           第四行：注释  
#           数据行：属性值  
#       repeated 表明下一个属性是repeated,即数组  
#           第二行: repeat的最大次数, excel中会重复列出该属性  
#           2011-11-29 做了修改 第二行如果是类型定义的话，则表明该列是repeated  
#           但是目前只支持整形  
#           第三行：无用  
#           第四行：注释  
#           数据行：实际的重复次数  
#       required_struct 必选结构属性  
#       optional_struct 可选结构属性  
#           第二行：结构元素个数  
#           第三行：结构名  
#           第四行：在上层结构中的属性名  
#           数据行：不用填  
  
#    1  | required/optional | repeated  | required_struct/optional_struct   |  
#       | ------------------| ---------:| ---------------------------------:|  
#    2  | 属性类型          |           | 结构元素个数                      |  
#    3  | 属性名            |           | 结构类型名                        |  
#    4  | 注释说明          |           | 在上层结构中的属性名              |  
#    5  | 属性值            |           |                                   |
```
当然可以参考github上下载到的样例表格，下载[tnt](https://github.com/jameyli/tnt/)项目，然后复制其中python目录下面的内容，其中xls文件中就有一个goods_info.xls的样例表格：
![](http://img.blog.csdn.net/20160821230930642)

#### 2.xls_deploy_tool.py转换得到.data和.proto：
进行导表的操作只需用在命令行中的一句指令即可完成：
```bash
python xls_deploy_tool.py sheet_name xls_path  
```
其中包含两个参数：sheet_name是.xls中要进行导表的表格页名，xls_path是要进行导表的.xls文件的路径。创建一个测试工程Test_protobuf，将1中的两个文件和protoc.exe放入其中：
![](http://img.blog.csdn.net/20160821231242376)
在命令行定位到该目录下，然后运行指令：
```bash
call python xls_deploy_tool.py GOODS_INFO xls/goods_info.xls  
```
运行结束后，该目录下多出了几个文件，但我们真正需要的只有两个文件，即.data数据文件和.proto解析类：
![](http://img.blog.csdn.net/20160821231633799)
![](http://img.blog.csdn.net/20160821231921863)

#### 3.得到最终解析类：
``protoc.exe``得到中间格式文件，假设后缀为.protodesc，使用指令：
```bash
protoc 输入文件路径(.proto文件) --descriptor_set_out=输出文件路径（.protodesc）  
```
在步骤2中的测试工程基础上继续执行指令：
```bash
protoc tnt_deploy_goods_info.proto --descriptor_set_out=goods_info.protodesc  
```
运行此步之后，在项目中又多出了一个与.proto对应的.protodesc文件：
![](http://img.blog.csdn.net/20160821232254537)  
protogen.exe得到.cs解析类，使用指令：
```bash
protogen -i:输入文件路径（.protodesc） -o:输出文件路径（.cs）  
```
将之前生成protogen.exe时protobuf-net-master\ProtoGen\bin\Release目录下面的所有文件复制到当前工程中，用一个文件夹ProtoGen来存放，假如不想执行这么繁琐的过程，也可以直接使用我编译好的ProtoGen文件目录压缩包：[ProtoGen.zip](http://download.csdn.net/detail/linshuhe1/9609654)，在当前项目的根目录下执行以下指令：
```bash
call ProtoGen\protogen -i:goods_info.protodesc -o:goods_info.cs  
```
执行结果，在当前目录下生成了解析类的最终状态goods_info.cs：
![](http://img.blog.csdn.net/20160822114317374)

当然，以上三步可以直接用批处理来完成，直接在当前项目根目录下新建一个文件，命名为generator.bat，内容为：
```bash
call python xls_deploy_tool.py GOODS_INFO xls/goods_info.xls  
call protoc tnt_deploy_goods_info.proto --descriptor_set_out=goods_info.protodesc  
call ProtoGen\protogen -i:goods_info.protodesc -o:goods_info.cs  
pause  
```
直接双击此文件即可完成以上所有操作生成最终的``.data``和``.cs``文件。
![](http://img.blog.csdn.net/20160822114656222)  

#### 4.Unity导入库文件：

将几个文件添加到Unity工程中，将.data文件放在Assets\StreamingAssets\DataConfig目录下，将protobuf-net.dll和goods_info.cs放在Assets目录下：
![](http://img.blog.csdn.net/20160822145729118)
![](http://img.blog.csdn.net/20160822145805294)
创建一个Test.cs测试脚本，在脚本中``using Protobuf``用于导入protobuf-net.dll中的库，然后使用``using tnt_deploy``导入导表生成的.cs表格数据解析类，脚本具体代码内容为：
```C#
using UnityEngine;  
using System.Collections;  
using ProtoBuf;  
using System.IO;  
using tnt_deploy;  
  
public class Test : MonoBehaviour {  
    void Start () {  
        GOODS_INFO_ARRAY goods_infos = ReadOneDataConfig<GOODS_INFO_ARRAY>("goods_info");  
        Debug.Log("goods_id==================" + goods_infos.items[0].goods_id);  
    }  
  
    private T ReadOneDataConfig<T>(string FileName)  
    {  
        FileStream fileStream;  
        fileStream = GetDataFileStream(FileName);  
        if (null != fileStream)  
        {  
            T t = Serializer.Deserialize<T>(fileStream);  
            fileStream.Close();  
            return t;  
        }  
  
        return default(T);  
    }  
    private FileStream GetDataFileStream(string fileName)  
    {  
        string filePath = GetDataConfigPath(fileName);  
        if (File.Exists(filePath))  
        {  
            FileStream fileStream = new FileStream(filePath, FileMode.Open);  
            return fileStream;  
        }  
  
        return null;  
    }  
    private string GetDataConfigPath(string fileName)  
    {  
        return Application.streamingAssetsPath + "/DataConfig/" + fileName + ".data";  
    }  
}  
```
在Unity中新建一个场景，将Test.cs挂载在Main Camera主相机上，运行场景，看到打印结果，说明解析表格数据成功：
![](http://img.blog.csdn.net/20160822145928497) 

#### 5.平台兼容问题：

由于直接把protobuf-net.dll放到项目中时，在iOS中会出现**JIT错误**（ExecutionEngineException: Attempting to JIT compile method）。原因是因为iOS不允许JIT（Just In Time），只允许AOT（Ahead Of Time）。

#### 解决方法：

直接把protprotobuf-net-master\protobuf-net目录下面的全部源码复制到Unity项目的目录下面，但是由于protobuf-net的编译过程是**unsafe编译**，所以Unity会出现编译报错：
![](http://img.blog.csdn.net/20160822151338719)
需要**在Assets目录下添加一个smsc.rsp文件**，其内容很简单，只有一行“-unsafe”，添加完成后关闭Unity然后重新打开Unity，一切就正常了。

### 四、总结：
虽然导表环境的配置过程比较繁琐，但是配置完成之后的工作效率却很高，而且proto具有突出的通用性，可以应用于各种语言环境。