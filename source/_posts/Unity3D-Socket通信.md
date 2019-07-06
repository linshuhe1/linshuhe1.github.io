---
title: Unity3D —— Socket通信
date: 2016-09-19 09:41:16
tags: 
  - Socket
  - C#
categories: Unity
---
### 前言：
在开始编写代码之前，我们首先需要明确：联网方式、联网步骤、数据收发以及协议数据格式

当然在设计时也应该减低代码的耦合性，尽量使得网络层可以在其他地方进行复用，这就需要我们进行接口式的开发。我们这里使用的通信模式是Socket强连接的通信方式，并且使用C#作为编程语言，其实与.NET的Socket通信是一致的。

### 一、设计思想：
为了方便测试，我直接使用C#写的一个控制台应用，作为服务器，等待客户端的连接，然后使用Unity建立Socket客户端去连接服务器，进行简单的数据通信。这么设计的原因是都基于.net进行开发，也方便理解。

<!--more-->

### 二、实现步骤：
对于网络通信有所了解的都应该知道，数据在网络传输的格式必须以字节流的形式进行，那么免不了要对字节流进行写入和读出操作，为了方便后面的操作，我们有必要封装一个读写字节流的操作类，在这里我定义了一个字节流的操作类ByteBuffer类，用于将各个类型数据写入流中，也可从字节流中读取各种类型的数据：
```C#
using System.IO;  
using System.Text;  
using System;  
  
namespace Net {  
    public class ByteBuffer {  
        MemoryStream stream = null;  
        BinaryWriter writer = null;  
        BinaryReader reader = null;  
  
        public ByteBuffer() {  
            stream = new MemoryStream();  
            writer = new BinaryWriter(stream);  
        }  
  
        public ByteBuffer(byte[] data) {  
            if (data != null) {  
                stream = new MemoryStream(data);  
                reader = new BinaryReader(stream);  
            } else {  
                stream = new MemoryStream();  
                writer = new BinaryWriter(stream);  
            }  
        }  
  
        public void Close() {  
            if (writer != null) writer.Close();  
            if (reader != null) reader.Close();  
  
            stream.Close();  
            writer = null;  
            reader = null;  
            stream = null;  
        }  
  
        public void WriteByte(byte v) {  
            writer.Write(v);  
        }  
  
        public void WriteInt(int v) {  
            writer.Write((int)v);  
        }  
  
        public void WriteShort(ushort v) {  
            writer.Write((ushort)v);  
        }  
  
        public void WriteLong(long v) {  
            writer.Write((long)v);  
        }  
  
        public void WriteFloat(float v) {  
            byte[] temp = BitConverter.GetBytes(v);  
            Array.Reverse(temp);  
            writer.Write(BitConverter.ToSingle(temp, 0));  
        }  
  
        public void WriteDouble(double v) {  
            byte[] temp = BitConverter.GetBytes(v);  
            Array.Reverse(temp);  
            writer.Write(BitConverter.ToDouble(temp, 0));  
        }  
  
        public void WriteString(string v) {  
            byte[] bytes = Encoding.UTF8.GetBytes(v);  
            writer.Write((ushort)bytes.Length);  
            writer.Write(bytes);  
        }  
  
        public void WriteBytes(byte[] v) {  
            writer.Write((int)v.Length);  
            writer.Write(v);  
        }  
  
        public byte ReadByte() {  
            return reader.ReadByte();  
        }  
  
        public int ReadInt() {  
            return (int)reader.ReadInt32();  
        }  
  
        public ushort ReadShort() {  
            return (ushort)reader.ReadInt16();  
        }  
  
        public long ReadLong() {  
            return (long)reader.ReadInt64();  
        }  
  
        public float ReadFloat() {  
            byte[] temp = BitConverter.GetBytes(reader.ReadSingle());  
            Array.Reverse(temp);  
            return BitConverter.ToSingle(temp, 0);  
        }  
  
        public double ReadDouble() {  
            byte[] temp = BitConverter.GetBytes(reader.ReadDouble());  
            Array.Reverse(temp);  
            return BitConverter.ToDouble(temp, 0);  
        }  
  
        public string ReadString() {  
            ushort len = ReadShort();  
            byte[] buffer = new byte[len];  
            buffer = reader.ReadBytes(len);  
            return Encoding.UTF8.GetString(buffer);  
        }  
  
        public byte[] ReadBytes() {  
            int len = ReadInt();  
            return reader.ReadBytes(len);  
        }  
  
        public byte[] ToBytes() {  
            writer.Flush();  
            return stream.ToArray();  
        }  
  
        public void Flush() {  
            writer.Flush();  
        }  
    }  
}  
```
使用此操作类进行读写数据的操作范例如下：

- 读取数据：
```C#
//result是字节数组byte[],从中读取两个int类型的数据  
 ByteBuffer buff = new ByteBuffer(result);  
 int len = buff.ReadShort();  
 int protoId = buff.ReadShort();  
```
- 写入数据：
```C#
//result是字节数组byte[],从写入两个不同类型的数据  
ByteBuffer buff = new ByteBuffer();  
int protoId = ProtoDic.GetProtoIdByProtoType(0);  
buff.WriteShort((ushort)protoId);  
buff.WriteBytes(ms.ToArray());  
byte[] result = buff.ToBytes();  
```

#### 1.服务器创建：
在VS中新建一个C#控制台应用，新建项目完成后将上面定义的ByteBuffer.cs类导入工程中，然后开始在入口类Program中开始创建Socket服务器的逻辑。

- 先引入必要的命名空间：
```C#
using System.Net;  
using System.Net.Sockets;  
using System.Threading;  
```
基本的步骤如下：

- 创建一个服务器Socket对象，并绑定服务器IP地址和端口号；
```C#
private const int port = 8088;  
private static string IpStr = "127.0.0.1";  
private static Socket serverSocket;  
  
static void Main(string[] args)  
{  
    IPAddress ip = IPAddress.Parse(IpStr);  
    IPEndPoint ip_end_point = new IPEndPoint(ip, port);  
    //创建服务器Socket对象，并设置相关属性  
    serverSocket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);  
    //绑定ip和端口  
    serverSocket.Bind(ip_end_point);  
    //设置最长的连接请求队列长度  
    serverSocket.Listen(10);  
    Console.WriteLine("启动监听{0}成功", serverSocket.LocalEndPoint.ToString());  
}  
```
完成上述代码之后，已经能正常启动一个服务器Socket，但是还没有处理连接监听逻辑和数据接收，所以运行应用会出现一闪就关掉的情况。

- 启动一个线程，并在线程中监听客户端的连接，为每个连接创建一个Socket对象；

- 创建接受数据和发送数据的方法，完整的代码如下：
```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Net.Sockets;  
using System.Text;  
using System.Threading.Tasks;  
using System.Net;  
using System.Threading;  
using Net;  
using System.IO;  
  
namespace ConsoleApplication1  
{  
    class Program  
    {  
        private static byte[] result = new byte[1024];  
        private const int port = 8088;  
        private static string IpStr = "127.0.0.1";  
        private static Socket serverSocket;  
  
        static void Main(string[] args)  
        {  
            IPAddress ip = IPAddress.Parse(IpStr);  
            IPEndPoint ip_end_point = new IPEndPoint(ip, port);  
            //创建服务器Socket对象，并设置相关属性  
            serverSocket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);  
            //绑定ip和端口  
            serverSocket.Bind(ip_end_point);  
            //设置最长的连接请求队列长度  
            serverSocket.Listen(10);  
            Console.WriteLine("启动监听{0}成功", serverSocket.LocalEndPoint.ToString());  
            //在新线程中监听客户端的连接  
            Thread thread = new Thread(ClientConnectListen);  
            thread.Start();  
            Console.ReadLine();  
        }  
  
        /// <summary>  
        /// 客户端连接请求监听  
        /// </summary>  
        private static void ClientConnectListen()  
        {  
            while (true)  
            {  
                //为新的客户端连接创建一个Socket对象  
                Socket clientSocket = serverSocket.Accept();  
                Console.WriteLine("客户端{0}成功连接", clientSocket.RemoteEndPoint.ToString());  
                //向连接的客户端发送连接成功的数据  
                ByteBuffer buffer = new ByteBuffer();  
                buffer.WriteString("Connected Server");  
                clientSocket.Send(WriteMessage(buffer.ToBytes()));  
                //每个客户端连接创建一个线程来接受该客户端发送的消息  
                Thread thread = new Thread(RecieveMessage);  
                thread.Start(clientSocket);  
            }  
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
  
        /// <summary>  
        /// 接收指定客户端Socket的消息  
        /// </summary>  
        /// <param name="clientSocket"></param>  
        private static void RecieveMessage(object clientSocket)  
        {  
            Socket mClientSocket = (Socket)clientSocket;  
            while (true)  
            {  
                try  
                {  
                    int receiveNumber = mClientSocket.Receive(result);  
                    Console.WriteLine("接收客户端{0}消息， 长度为{1}", mClientSocket.RemoteEndPoint.ToString(), receiveNumber);  
                    ByteBuffer buff = new ByteBuffer(result);  
                    //数据长度  
                    int len = buff.ReadShort();  
                    //数据内容  
                    string data = buff.ReadString();  
                    Console.WriteLine("数据内容：{0}", data);  
                }  
                catch (Exception ex)  
                {  
                    Console.WriteLine(ex.Message);  
                    mClientSocket.Shutdown(SocketShutdown.Both);  
                    mClientSocket.Close();  
                    break;  
                }  
            }  
        }  
    }  
}  
```
#### 2.客户端创建：
客户端连接服务器的逻辑相对简单一些，跟服务器一样，先把ByteBuffer类导入到工程中，基本步骤如下：

- 创建一个Socket对象，这个对象在客户端是唯一的，可以理解为单例模式；
- 使用上面创建Socket连接指定服务器IP和端口号；
- 接收服务器数据和发送数据给服务器。

先创建一个ClientSocket类用于管理Socket的一些方法：连接服务器、接受数据和发送数据等：
```C#
using UnityEngine;  
using System.Collections;  
using System.Net;  
using System.Net.Sockets;  
using System.IO;  
  
namespace Net  
{  
    public class ClientSocket  
    {  
        private static byte[] result = new byte[1024];  
        private static Socket clientSocket;  
        //是否已连接的标识  
        public bool IsConnected = false;  
  
        public ClientSocket(){  
            clientSocket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);  
        }  
  
        /// <summary>  
        /// 连接指定IP和端口的服务器  
        /// </summary>  
        /// <param name="ip"></param>  
        /// <param name="port"></param>  
        public void ConnectServer(string ip,int port)  
        {  
            IPAddress mIp = IPAddress.Parse(ip);  
            IPEndPoint ip_end_point = new IPEndPoint(mIp, port);  
  
            try {  
                clientSocket.Connect(ip_end_point);  
                IsConnected = true;  
                Debug.Log("连接服务器成功");  
            }  
            catch  
            {  
                IsConnected = false;  
                Debug.Log("连接服务器失败");  
                return;  
            }  
            //服务器下发数据长度  
            int receiveLength = clientSocket.Receive(result);  
            ByteBuffer buffer = new ByteBuffer(result);  
            int len = buffer.ReadShort();  
            string data = buffer.ReadString();  
            Debug.Log("服务器返回数据：" + data);  
        }  
  
        /// <summary>  
        /// 发送数据给服务器  
        /// </summary>  
        public void SendMessage(string data)  
        {  
            if (IsConnected == false)  
                return;  
            try  
            {  
                ByteBuffer buffer = new ByteBuffer();  
                buffer.WriteString(data);  
                clientSocket.Send(WriteMessage(buffer.ToBytes()));  
            }  
            catch  
            {  
                IsConnected = false;  
                clientSocket.Shutdown(SocketShutdown.Both);  
                clientSocket.Close();  
            }  
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
    }  
}  
```

### 三、样例测试：
#### 1.客户端测试：
在Unity中写一个测试脚本TestSocket.cs，并将此脚本绑到当前场景的相机上：
```C#
using UnityEngine;  
using System.Collections;  
using Net;  
  
public class TestSocket : MonoBehaviour {  
  
    // Use this for initialization  
    void Start () {  
        ClientSocket mSocket = new ClientSocket();  
        mSocket.ConnectServer("127.0.0.1", 8088);  
        mSocket.SendMessage("服务器傻逼！");  
    }  
      
    // Update is called once per frame  
    void Update () {  
      
    }  
}  
```
#### 2.启动服务器：
在Visual Studio中点击运行按钮，启动服务器：
![](http://img.blog.csdn.net/20160825164018638)
启动正常的话，会弹出一个窗口如下图所示：
![](http://img.blog.csdn.net/20160825164121342)

#### 3.开始连接：
在Unity中运行当前场景，查看输出日志，假如连接成功，输出如下：
![](http://img.blog.csdn.net/20160825164333224)
查看服务器窗口，发现双向通信都正常：
![](http://img.blog.csdn.net/20160825164551251)

### 四、总结：
这里测试案例其实很简单，协议没有进行如何优化，单纯地发送字符串数据而已，假如针对复杂的数据的话，需要创建完整打包和解包协议数据的机制，而且必要时还需要对数据进行加密操作。