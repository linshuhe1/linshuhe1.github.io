---
title: Unity3D —— protobuf网络框架
date: 2016-09-19 09:56:03
tags: 
  - protobuf
  - Unity
categories: Unity
---
### 前言：
protobuf是google的一个开源项目，主要的用途是：

- 数据存储（序列化和反序列化），这个功能类似xml和json等；
- 制作网络通信协议；

### 一、资源下载：

- github源码地址：[protobuf-net](https://github.com/mgravell/protobuf-net)
- google项目源码下载地址（**访问需翻墙**）：[protobuf-net](https://code.google.com/p/protobuf-net/)

<!--more-->

### 二、数据存储：
C#语言方式的导表和解析过程，在之前的篇章中已经有详细的阐述：[Unity —— protobuf 导excel表格数据](http://blog.csdn.net/linshuhe1/article/details/52062969)，建议在看后续的操作之前先看一下这篇文档，因为后面设计到得一些操作与导表中是一致的，而且在理解了导表过程之后，能够快速地理解协议数据**序列化**和**反序列化**的过程。

### 三、网络协议：
#### 1.设计思想：
有两个必要的数据：**协议号**和**协议类型**，将这两个数据分别存储起来

- 当客户端向服务器发送数据时，会根据协议类型加上协议号，然后使用protobuf序列化之后再发送给服务器；
- 当服务器发送数据给客户端时，根据协议号，用protobuf根据协议类型反序列化数据，并调用相应回调方法。

由于数据在传输过程中，都是以数据流的形式存在的，而进行解析时无法单从protobuf数据中得知使用哪个解析类进行数据反序列化，这就要求我们在传输protobuf数据的同时，携带一个协议号，通过协议号和协议类型（解析类）之间的对应关系来确定进行数据反序列化的解析类。
![](http://img.blog.csdn.net/20160824100320406)    
此处协议号的作用就是用来确定用于解析数据的解析类，所以也可能称之为**协议类型名**，可以是``string``和``int``类型的数据。

#### 2.特点分析：
使用protobuf作为网络通信的数据载体，具有几个优点：

- 通过序列化之后**数据量比较小**；
- 而且**以key-value的方式存储数据**，这对于消息的版本兼容比较强；
- 此外，由于protobuf提供的多语言支持，所以使用protobuf作为数据载体定制的网络协议**具有很强的跨语言特性**。

### 四、样例实现：
#### 1.协议定义：
在之前导表的时候，我们得到了.proto的解析类，这是protobuf提供的一种特殊的脚本，具有格式简单、可读性强和方便拓展的特点，所以接下来我们就是**使用proto脚本来定义我们的协议**。例如：
```C#
// 物品  
message Item  
{  
    required int32 Type     = 1;    //游戏物品大类  
    optional int32 SubType  = 2;    //游戏物品小类  
    required int32 num      = 3;    //游戏物品数量  
}  
  
// 物品列表  
message ItemList  
{  
    repeated Item item  = 1;    //物品列表  
}  
```
上述例子中，Item相当于定义了一个数据结构或者是类，而ItemList是一个列表，列表中的每个元素都是一个Item对象。注意结构关键词：

- ``required``：必有的属性
- ``optional``：可选属性
- ``repeated``：数组

其实protobuf在这里只是提供了一个数据载体，通过在.proto中定义数据结构之后，需要使用与导表时一样的操作，步骤为：

- 使用**protoc.exe**将.proto文件转化为.protodesc中间格式；
- 使用**protogen.exe**将中间格式为.protodesc生成指定的高级语言类，我们在Unity中使用的是C#,所以结果是.cs类

经过上述步骤之后，我们得到了协议类型对应的C#反序列化类，当我们收到服务器数据时，**根据协议号找到协议类型**，从而使用对应的反序列化的类对数据进行反序列化，得到最终的服务器数据内容。

在这里，我们以登录为例，首先要清楚登录需要几个数据，正常情况下至少包含两个数据，即账号和密码，都是字符串类型，即定义cs_login.proto协议脚本，内容如下：
```C#
package cs;  
  
message CSLoginInfo  
{  
    required string UserName = 1;//账号  
    required string Password = 2;//密码  
}  
  
//发送登录请求  
message CSLoginReq  
{  
    required CSLoginInfo LoginInfo = 1;   
}  
//登录请求回包数据  
message CSLoginRes  
{  
    required uint32 result_code = 1;   
}  
```
``package``关键字后面的名称为.proto转为.cs之后的命名空间namespace的值，用message可以定义类，这里定义了一个CSLoginInfo的数据类，该类包含了账号和密码两个字符串类型的属性。然后定义了两个消息结构：

- CSLoginReq登录请求消息，携带的数据是一个CSLoginInfo类型的对象数据；
- CSLoginRes登录请求服务器返回的数据类型，返回结果是一个uint32无符号的整型数据，即结果码。

上面定义的是协议类型，除此之外我们还需要为每一个协议类型定义一个协议号，这里可以用一个枚举脚本cs_enum.proto来保存，脚本内容为：
```C#
package cs;  
  
enum EnmCmdID  
{  
    CS_LOGIN_REQ = 10001;//登录请求协议号  
    CS_LOGIN_RES = 10002;//登录请求回包协议号  
}  
```
使用protoc.exe和protogen.exe将这两个protobuf脚本得到C#类，具体步骤参考导表使用的操作，这里我直接给出自动化导表使用的批处理文件general_all.bat内容，具体文件目录可以根据自己放置情况进行调整：
```bash
::---------------------------------------------------  
::第二步：把proto翻译成protodesc  
::---------------------------------------------------  
call proto2cs\protoc protos\cs_login.proto --descriptor_set_out=cs_login.protodesc  
call proto2cs\protoc protos\cs_enum.proto --descriptor_set_out=cs_enum.protodesc  
::---------------------------------------------------  
::第二步：把protodesc翻译成cs  
::---------------------------------------------------  
call proto2cs\ProtoGen\protogen -i:cs_login.protodesc -o:cs_login.cs  
call proto2cs\ProtoGen\protogen -i:cs_enum.protodesc -o:cs_enum.cs  
::---------------------------------------------------  
::第二步：把protodesc文件删除  
::---------------------------------------------------  
del *.protodesc  
  
pause  
```
转换结束后，我们的得到了两个.cs文件分别是：cs_enum.cs和cs_login.cs，将其放入到我们的Unity项目中，以便于接下来序列化和反序列化数据的使用。

#### 2.协议数据构建：
直接在项目代码中通过``using cs``引入协议解析类的命名空间，然后创建消息对象，并对对象的属性进行赋值，即可得到协议数据对象，例如登录请求对象的创建如下：
```C#
CSLoginInfo mLoginInfo = new CSLoginInfo();  
mLoginInfo.UserName = "linshuhe";  
mLoginInfo.Password = "123456";  
CSLoginReq mReq = new CSLoginReq();  
mReq.LoginInfo = mLoginInfo;  
```
从上述代码，可以得到登录请求对象mReq，里面包含了一个CSLoginInfo对象mLoginInfo，再次枚举对象中找到与此协议类型对应的协议号，即：``EnmCmdID.CS_LOGIN_REQ``

#### 3.数据的序列化和反序列化：
数据发送的时候必须以数据流的形式进行，所以这里我们需要考虑如何**将要发送的protobuf对象数据进行序列化，转化为byte[]字节数组**，这就需要借助ProtoBuf库为我们提供的``Serializer``类的``Serialize``方法来完成，而反序列化则需借助``Deserialize``方法，将这两个方法封装到PackCodec类中：
```C#
using UnityEngine;  
using System.Collections;  
using System.IO;  
using System;  
using ProtoBuf;  
  
/// <summary>  
/// 网络协议数据打包和解包类  
/// </summary>  
public class PackCodec{  
    /// <summary>  
    /// 序列化  
    /// </summary>  
    /// <typeparam name="T"></typeparam>  
    /// <param name="msg"></param>  
    /// <returns></returns>  
    static public byte[] Serialize<T>(T msg)  
    {  
        byte[] result = null;  
        if (msg != null)  
        {  
            using (var stream = new MemoryStream())  
            {  
                Serializer.Serialize<T>(stream, msg);  
                result = stream.ToArray();  
            }  
        }  
        return result;  
    }  
  
    /// <summary>  
    /// 反序列化  
    /// </summary>  
    /// <typeparam name="T"></typeparam>  
    /// <param name="message"></param>  
    /// <returns></returns>  
    static public T Deserialize<T>(byte[] message)  
    {  
        T result = default(T);  
        if (message != null)  
        {  
            using (var stream = new MemoryStream(message))  
            {  
                result = Serializer.Deserialize<T>(stream);  
            }  
        }  
        return result;  
    }  
}  
```
使用方法很简单，直接传入一个数据对象即可得到字节数组：
```C#
byte[] buf = PackCodec.Serialize(mReq);  
```
为了检验打包和解包是否匹配，我们可以直接做一次本地测试：将打包后的数据直接解包，看看数据是否与原来的一致：
```C#
using UnityEngine;  
using System.Collections;  
using System;  
using cs;  
using ProtoBuf;  
using System.IO;  
  
public class TestProtoNet : MonoBehaviour {  
  
    // Use this for initialization  
    void Start () {  
        CSLoginInfo mLoginInfo = new CSLoginInfo();  
        mLoginInfo.UserName = "linshuhe";  
        mLoginInfo.Password = "123456";  
        CSLoginReq mReq = new CSLoginReq();  
        mReq.LoginInfo = mLoginInfo;  
  
        byte[] pbdata = PackCodec.Serialize(mReq);  
        CSLoginReq pReq = PackCodec.Deserialize<CSLoginReq>(pbdata);  
        Debug.Log("UserName = " + pReq.LoginInfo.UserName + ", Password = " + pReq.LoginInfo.Password);  
    }  
  
    // Update is called once per frame  
    void Update () {  
      
    }  
} 
```
将此脚本绑到场景中的相机上，运行得到以下结果，则说明打包和解包完全匹配：
![](http://img.blog.csdn.net/20160825100033496)    

#### 4.数据发送和接收：
这里我们使用的网络通信方式是Socket的强联网方式，关于如何在Unity中使用Socket进行通信，可以参考我之前的文章：[Unity —— Socket通信(C#)](http://blog.csdn.net/linshuhe1/article/details/51386559)，Unity客户端需要复制此项目的**ClientSocket.cs**和**ByteBuffer.cs**两个类到当前项目中。

此外，服务器可以参照之前的方式搭建，唯一不同的是RecieveMessage(object clientSocket)方法解析数据的过程需要进行修改，因为需要使用protobuf-net.dll进行数据解包，所以需要参考客户端的做法，把protobuf-net.dll复制到服务器项目中的Protobuf_net目录下：  
![](http://img.blog.csdn.net/20160825175754877)
假如由于直接使用源码而不用.dll会出现不安全保存，需要在Visual Studio中设置允许不安全代码，具体步骤为：在“解决方案”中选中工程，右键“数据”，选择“生成”页签，勾选“允许不安全代码”：
![](http://img.blog.csdn.net/20160825180841975)    
当然，解析数据所用的解析类和协议号两个脚本cs_login.cs和cs_enum.cs也应该添加到服务器项目中，保证客户端和服务器一直，此外PackCodec.cs也需要添加到服务器代码中但是要把其中的using UnityEngine给去掉防止报错，最终服务器目录结构如下：
![](http://img.blog.csdn.net/20160825191847662)

#### 5.完整协议数据的封装：
从之前说过的设计思路分析，我们在发送数据的时候除了要发送关键的protobuf数据之外，还需要带上两个附件的数据：协议头（用于进行通信检验）和协议号（用于确定解析类）。假设我们的是：

- **协议头**：用于表示后面数据的长度，一个short类型的数据：
```C#
/// <summary>  
/// 数据转换，网络发送需要两部分数据，一是数据长度，二是主体数据  
/// </summary>  
/// <param name="message"></param>  
/// <returns></returns>  
private static byte[] WriteMessage(byte[] message)  
{  
    MemoryStream ms = null;  
    using (ms = new MemoryStream())  
    {  
        ms.Position = 0;  
        BinaryWriter writer = new BinaryWriter(ms);  
        ushort msglen = (ushort)message.Length;  
        writer.Write(msglen);  
        writer.Write(message);  
        writer.Flush();  
        return ms.ToArray();  
    }  
}  
```
- **协议号**：用于对应解析类，这里我们使用的是int类型的数据：

```C#
private byte[] CreateData(int typeId,IExtensible pbuf)  
  
byte[] pbdata = PackCodec.Serialize(pbuf);  
ByteBuffer buff = new ByteBuffer();  
buff.WriteInt(typeId);  
buff.WriteBytes(pbdata);  
return buff.ToBytes();  
```

客户端发送登录数据时测试脚本TestProtoNet如下，测试需要将此脚本绑定到当前场景的相机上：

```C#
using UnityEngine;  
using System.Collections;  
using System;  
using cs;  
using Net;  
using ProtoBuf;  
using System.IO;  
  
public class TestProtoNet : MonoBehaviour {  
  
    // Use this for initialization  
    void Start () {  
  
  
        CSLoginInfo mLoginInfo = new CSLoginInfo();  
        mLoginInfo.UserName = "linshuhe";  
        mLoginInfo.Password = "123456";  
        CSLoginReq mReq = new CSLoginReq();  
        mReq.LoginInfo = mLoginInfo;  
  
        byte[] data = CreateData((int)EnmCmdID.CS_LOGIN_REQ, mReq);  
        ClientSocket mSocket = new ClientSocket();  
        mSocket.ConnectServer("127.0.0.1", 8088);  
        mSocket.SendMessage(data);  
    }  
  
    private byte[] CreateData(int typeId,IExtensible pbuf)  
    {  
        byte[] pbdata = PackCodec.Serialize(pbuf);  
        ByteBuffer buff = new ByteBuffer();  
        buff.WriteInt(typeId);  
        buff.WriteBytes(pbdata);  
        return WriteMessage(buff.ToBytes());  
    }  
  
    /// <summary>  
    /// 数据转换，网络发送需要两部分数据，一是数据长度，二是主体数据  
    /// </summary>  
    /// <param name="message"></param>  
    /// <returns></returns>  
    private static byte[] WriteMessage(byte[] message)  
    {  
        MemoryStream ms = null;  
        using (ms = new MemoryStream())  
        {  
            ms.Position = 0;  
            BinaryWriter writer = new BinaryWriter(ms);  
            ushort msglen = (ushort)message.Length;  
            writer.Write(msglen);  
            writer.Write(message);  
            writer.Flush();  
            return ms.ToArray();  
        }  
    }  
  
    // Update is called once per frame  
    void Update () {  
      
    }  
} 
```

服务器接受数据解包过程参考打包数据的格式，在RecieveMessage(object clientSocket)中，解析数据的核心代码如下：

```C#
ByteBuffer buff = new ByteBuffer(result);  
int datalength = buff.ReadShort();  
int typeId = buff.ReadInt();  
byte[] pbdata = buff.ReadBytes();  
//通过协议号判断选择的解析类  
if(typeId == (int)EnmCmdID.CS_LOGIN_REQ)  
{  
        CSLoginReq clientReq = PackCodec.Deserialize<CSLoginReq>(pbdata);  
        string user_name = clientReq.LoginInfo.UserName;  
        string pass_word = clientReq.LoginInfo.Password;  
        Console.WriteLine("数据内容：UserName={0},Password={1}", user_name, pass_word);  
        }  
}  
```
上面通过typeId来找到匹配的数据解析类，协议少的时候可以使用这种简单的使用if语句分支判断来实现，但是假如协议类型多了，则需要进一步封装查找方法，常用的方法有：定义一个Dictionary<int,Type>字典来存放协议号（int）和协议类型（Type）的对应关系。

#### 6.运行结果：
启动服务器，然后运行Unity中的客户端，得到正确的结果应该如下：
![](http://img.blog.csdn.net/20160825192659728)
项目服务器和客户端的完整代码可以前往此处下载：[protobuf-net网络协议的定制](http://download.csdn.net/detail/linshuhe1/9613076)