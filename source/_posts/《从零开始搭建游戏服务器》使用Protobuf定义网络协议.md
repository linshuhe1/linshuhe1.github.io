---
title: 《从零开始搭建游戏服务器》使用Protobuf定义网络协议
date: 2017-03-03 10:28:37
tags: 
  - Java
  - Protobuf
categories: 服务器开发
---

>引言：
之前我们说过使用Protobuff编解码方式来对网络通信数据进行编码，也就是可以用这种方式来定义通信协议，那么具体要怎么做呢？关于此工具的详细资料和使用操作可以参考我之前写过的博客：[Unity3D —— protobuf网络框架](http://blog.csdn.net/linshuhe1/article/details/51781749)

<!--more-->
---

### 准备工具：
这里我就直接用之前编写的登录案例中的登录协议文件来进行本次案例的测试，工具下载地址如下：[protobuf-java-2.5.0.zip](http://download.csdn.net/detail/yangheng362/8516923)，主要使用到的文件有：
- **protoc.exe**工具：通过此工具将从自定义的协议文件（.proto）得到相应（.java）的Java类文件；
- 对应proto.exe版本的**protobuf-java.jar**包，用于解析上面得到的.java类，这里我使用的是2.5.0版本的protobuf；
- cs_login.proto协议文件，关于proto协议文件的书写语法详细的可以查看：[Protobuf语言指南](http://www.cnblogs.com/dkblog/archive/2012/03/27/2419010.html)，cs_login.proto内容如下（包名package可以根据当前服务器应用的包名进行修改）：
    ```java
    package com.tw.login.proto;

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
	cs_enum.proto协议类型枚举文件，用于列举所有协议数据结构的编号：
    ```java
    package com.tw.login.proto;

    enum EnmCmdID
    {
        CS_LOGIN_REQ = 10001;//登录请求协议号
        CS_LOGIN_RES = 10002;//登录请求回包协议号
    }
	```
- 将.proto转化为Java类文件的处理脚本，这里其实只是一句命令行指令：
	```
	protoc --java_out=(.java输出目录) (.proto文件存放目录)\cs_login.proto
	```
	这里我根据实际项目目录结构定义了一个转文件的.bat批处理文件：
    ```
    echo on
    call protoc --version

    call protoc --java_out=..\src\ cs_login.proto
    call protoc --java_out=..\src\ cs_enum.proto

    PAUSE
    ```
    tool文件夹在当前java工程的根目录下，其下有三种文件：.proto的协议文件、转化文件用的protoc.exe工具和批处理文件general_java.bat：
    ![](http://img.blog.csdn.net/20161220205058824)

---
### 在Netty服务器验证：
####1. 转化协议：
双击打开general_java.bat，转.proto协议得到.java脚本，转化成功现象如下，并且在package指定的目录下生产对应的.java类：
	![](http://img.blog.csdn.net/20161220205533580)
    ![](http://img.blog.csdn.net/20161220205648112)

####2.修改对通道**ChannelPipeline**中编解码格式的设置：
 有一定网络通信协议基础的应该知道（不知道的赶紧补一下）：应用层位于传输层之上，当应用层要从传输层的Socket读取数据时，是从下向上层的传输，也就是``Upstream``；而反过来，从应用层向传输层的Socket写入并发数据时，则为``Downstream``。
 Upstream 和 Downstream 都是在 Pipeline 中“流动”的，所以影响 Upstream 和 Downstream 行为的 UpstreamHandler 和 DownstreamHandler，也要被放到 Pipeline 里。了解了上述这两个概念之后，我们就可以开始修改管道ChannelPipeline的参数，包括：
    - ``frameDecoder``和``protobufDecoder``对应的handler用于解码Protobuf package数据包，他们都是Upstream Handles；
    - ``frameEncoder``和``protobufEncoder``对应的handler用于编码Protobuf package数据包，他们都是Downstream Handles；
    - 此外还有一个handler，是一个自定义的Upstream Handles，**用于开发者从网络数据中解析得到自己所需的数据**。

 添加到Pipeline中的handler其实有特定的执行顺序，例如：
    ```java
	pipeline.addLast("frameDecoder",
						new ProtobufVarint32FrameDecoder());
	pipeline.addLast("protobufDecoder", new ProtobufDecoder(CsLogin.CSLoginReq.getDefaultInstance()));
	pipeline.addLast("frameEncoder",new ProtobufVarint32LengthFieldPrepender());
	pipeline.addLast("protobufEncoder", new ProtobufEncoder());
	pipeline.addLast("handler",new SocketServerHandler());
    ```
 上例的执行顺序为：
	```java
    upstream:frameDecoder,protobufDecoder,handler 	//解码从Socket收到的数据
    downstream:frameEncoder,protobufEncoder			//编码要通过Socket发送出去的数据
    ```
 这里我们假设协议数据的格式为纯protobuf数据格式，不进行如何加密操作，需要注意的一点：**protobufDecoder仅仅负责编码，并不支持读半包**，所以在之前一定要有**读半包的处理器**，即这里用到的frameDecoder，通常用此处理器读取包的数据长度，有三种方式可以选择：
	- 使用netty提供的 ``ProtobufVarint32FrameDecoder``；
	- 继承netty提供的通用半包处理器 ``LengthFieldBasedFrameDecoder``；
	- 继承netty提供的通用半包处理器 ``LengthFieldBasedFrameDecoder``。
---
### protobuf数据通信过程：
####1.客户端创建数据：
要构建一个protobuf数据，需要通过对应协议文件的数据结构，先通过每个数据类型的newBuilder()方法来创建对应的Builder对象，再对Builder中的属性进行赋值，最后才能使用Builder来build()数据对象，在客户端的自定义handler控制器``SocketClientHandler``中的**channelActive**方法中发送数据（注：当客户端和服务端建立tcp成功之后，Netty的NIO线程会调用channelActive）：
```java
@Override
public void channelActive(ChannelHandlerContext ctx) {
	CSLoginReq.Builder req_builder = CSLoginReq.newBuilder();
    CSLoginInfo.Builder info_builder = CSLoginInfo.newBuilder();
    info_builder.setUserName("linshuhe");
    info_builder.setPassword("123456");
    CSLoginInfo info = info_builder.build();
    req_builder.setLoginInfo(info);
    CSLoginReq req = req_builder.build();
    ctx.writeAndFlush(req);
}
```
最后，需要将数据对象通过writeAndFlush()发送出去用Socket传输。
####2.服务器解析数据：
在服务器自定义的handler控制器``SocketServerHandler``的**ChannelRead**方法中进行数据解析：
```java
public void channelRead(ChannelHandlerContext arg0, Object msg) throws Exception {
	// TODO Auto-generated method stub
	CSLoginReq clientReq = (CSLoginReq)msg;
    String user_name = clientReq.getLoginInfo().getUserName();
    String pass_word = clientReq.getLoginInfo().getPassword();
    logger.info("数据内容：UserName="+user_name+",Password="+pass_word);
}
```
####3.执行结果：
服务器打印输出结果如下：
![](http://img.blog.csdn.net/20170221133626442)

---
####备注：
假如自定义的handler出现接受数据时只执行``channelReadComplete``方法而不执行``ChannelRead``方法，则说明数据发送的格式不对，因为只执行channelReadComplete说明收到消息没有能够被frameDecoder识别的指定的结束标志，例如：frameDecoder设置为lineBasedFrameDecoder时，结束标志为换行符。
