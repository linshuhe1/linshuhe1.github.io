---
title: 《从零开始搭建游戏服务器》多个Protobuf协议的改造
date: 2017-03-03 10:38:37
tags: 
  - Java
  - Protobuf
categories: 服务器开发
---

>### 引言
通过上篇 [《从零开始搭建游戏服务器》使用Protobuf定义网络协议](http://blog.csdn.net/linshuhe1/article/details/56280797) 的实践之后，我们知道在设置ChannelPiple的handler时，只能设置一个解码器，即protobufDecoder，但是在实际的网络通信过程中，我们需要传输的数据类型必然有很多种，这就需要通过某个标识来帮助我们在进行数据解析时进行数据类型的判断。

<!--more-->

### 解决方案：
#### 方案一
这是protobuf官方的一种实现方式，通过自定义解析类，将要进行传输的protobuf类型数据都先转换为字节数组，放在一个byte数组里面，定义额外的属性来描述对应的proto文件和协议数据类型，格式如下:

```
message SelfDescribingMessage {
  // Set of .proto files which define the type.
  required FileDescriptorSet proto_files = 1;

  // Name of the message type.  Must be defined by one of the files in
  // proto_files.
  required string type_name = 2;

  // The message data.
  required bytes message_data = 3;
}
```
 - ##### 优点：
如此，信道上传输的实际类型就是SelfDescribingMessage类型，由于message_data的兼容性，此数据类型可以承载任何类型的数据。
 - ##### 缺点：
由于每次传输数据，序列化步骤包括：普通数据类型序列化为protobuf类型，protobuf类型序列化为byte数组，反序列化也是一个逆过程，如此就需要进行两次序列化和两次反序列化，这对于那些对延迟和性能比较敏感的系统，显然不是一个好的选择。

#### 方案二
直接在protobuf序列化数据的前面，加上一个自定义的协议头，协议头里包含序列数据的长度和对应的数据类型，在数据解包的时候根据包头来进行反序列化。

##### 协议头定义
关于这一块，我打算先采取比较简单的办法，结构如下：
![](http://img.blog.csdn.net/20160824100320406)
协议号是自定义的一个``int``类型的枚举（当然，假如协议吧比较少的话，可以用一个``short``来代替int以缩小数据包），这个协议号与协议类型是一一对应的，而协议头通常使用数据总长度来填入，具体过程如下：
 - 当客户端向服务器发送数据时，会根据协议类型加上协议号，然后使用protobuf序列化之后再发送给服务器；
 - 当服务器发送数据给客户端时，根据协议号，确定protobuf协议类型以反序列化数据，并调用相应回调方法。

这个办法是我之前在C#中尝试过的，具体情况可以参考：[Unity3D —— protobuf网络框架](http://blog.csdn.net/linshuhe1/article/details/51781749)。

##### 自定义的编码器和解码器
 - **编码器**：
参考netty自带的编码器``ProtobufEncoder``可以发现，被绑定到ChannelPipeline上用于序列化协议数据的编码器，必须继承``MessageToByteEncoder<MessageLite>``这个基类，并通过重写``protected void encode(ChannelHandlerContext ctx, MessageLite msg, ByteBuf out)``这个方法来实现自定义协议格式的目的：

    ```java
    package com.tw.login.tools;

    import com.google.protobuf.MessageLite;
    import com.tw.login.proto.CsEnum.EnmCmdID;
    import com.tw.login.proto.CsLogin;

    import io.netty.buffer.ByteBuf;
    import io.netty.channel.ChannelHandlerContext;
    import io.netty.handler.codec.MessageToByteEncoder;
    /**
     * 自定义编码器
     * @author linsh
     *
     */
    public class PackEncoder extends MessageToByteEncoder<MessageLite> {

        /**
         * 传入协议数据，产生携带包头之后的数据
         */
        @Override
        protected void encode(ChannelHandlerContext ctx, MessageLite msg, ByteBuf out) throws Exception {
            // TODO Auto-generated method stub
            byte[] body = msg.toByteArray();
            byte[] header = encodeHeader(msg, (short)body.length);

            out.writeBytes(header);
            out.writeBytes(body);
            return;
        }

        /**
         * 获得一个协议头
         * @param msg
         * @param bodyLength
         * @return
         */
        private byte[] encodeHeader(MessageLite msg,short bodyLength){
            short _typeId = 0;
            if(msg instanceof CsLogin.CSLoginReq){
                _typeId = EnmCmdID.CS_LOGIN_REQ_VALUE;
            }else if(msg instanceof CsLogin.CSLoginRes){
                _typeId = EnmCmdID.CS_LOGIN_RES_VALUE;
            }
			//存放两个short数据
            byte[] header = new byte[4];
            //前两位放数据长度
            header[0] = (byte) (bodyLength & 0xff);
            header[1] = (byte) ((bodyLength >> 8) & 0xff);
            //后两个字段存协议id
            header[2] = (byte) (_typeId & 0xff);
            header[3] = (byte) ((_typeId >> 8) & 0xff);
            return header;
        }
    }
    ```

 - **解码器**：
参考netty自带的编码器``ProtobufDecoder``可以发现，被绑定到ChannelPipeline上用于序列化协议数据的解码器，必须继承``ByteToMessageDecoder``这个基类，并通过重写``protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out)``这个方法来实现解析自定义协议格式的目的：

	```java
    package com.tw.login.tools;

    import java.util.List;

    import com.google.protobuf.MessageLite;
    import com.tw.login.proto.CsEnum.EnmCmdID;
    import com.tw.login.proto.CsLogin.CSLoginReq;
    import com.tw.login.proto.CsLogin.CSLoginRes;

    import io.netty.buffer.ByteBuf;
    import io.netty.channel.ChannelHandlerContext;
    import io.netty.handler.codec.ByteToMessageDecoder;

    public class PackDecoder extends ByteToMessageDecoder {

        @Override
        protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
            // 获取包头中的body长度
            byte low = in.readByte();
            byte high = in.readByte();
            short s0 = (short) (low & 0xff);
            short s1 = (short) (high & 0xff);
            s1 <<= 8;
            short length = (short) (s0 | s1);

            // 获取包头中的protobuf类型
            byte low_type = in.readByte();
            byte high_type = in.readByte();
            short s0_type = (short) (low_type & 0xff);
            short s1_type = (short) (high_type & 0xff);
            s1_type <<= 8;
            short dataTypeId = (short) (s0_type | s1_type);

            // 如果可读长度小于body长度，恢复读指针，退出。
            if (in.readableBytes() < length) {
                in.resetReaderIndex();
                return;
            }

            //开始读取核心protobuf数据
            ByteBuf bodyByteBuf = in.readBytes(length);
            byte[] array;
            //反序列化数据的起始点
            int offset;
            //可读的数据字节长度
            int readableLen= bodyByteBuf.readableBytes();
            //分为包含数组数据和不包含数组数据两种形式
            if (bodyByteBuf.hasArray()) {
                array = bodyByteBuf.array();
                offset = bodyByteBuf.arrayOffset() + bodyByteBuf.readerIndex();
            } else {
                array = new byte[readableLen];
                bodyByteBuf.getBytes(bodyByteBuf.readerIndex(), array, 0, readableLen);
                offset = 0;
            }

            //反序列化
            MessageLite result = decodeBody(dataTypeId, array, offset, readableLen);
            out.add(result);
        }

        /**
         * 根据协议号用响应的protobuf类型来解析协议数据
         * @param _typeId
         * @param array
         * @param offset
         * @param length
         * @return
         * @throws Exception
         */
        public MessageLite decodeBody(int _typeId,byte[] array,int offset,int length) throws Exception{
            if(_typeId == EnmCmdID.CS_LOGIN_REQ_VALUE){
                return CSLoginReq.getDefaultInstance().getParserForType().parseFrom(array,offset,length);
            }
            else if(_typeId == EnmCmdID.CS_LOGIN_RES_VALUE){
                return CSLoginRes.getDefaultInstance().getParserForType().parseFrom(array,offset,length);
            }
            return null;
        }
    }
   ```
   
#### 修改Socket管道绑定的编解码器：
在创建Socket管道的时候，将编解码器替换为自定义的编解码器，而具体数据发送和接受过程无需做任何修改：
```java
ChannelPipeline pipeline = ch.pipeline();
// 协议数据的编解码器
pipeline.addLast("frameDecoder",new ProtobufVarint32FrameDecoder());
pipeline.addLast("protobufDecoder",new PackDecoder());
pipeline.addLast("frameEncoder",new ProtobufVarint32LengthFieldPrepender());
pipeline.addLast("protobufEncoder", new PackEncoder());
pipeline.addLast("handler",new SocketServerHandler());
```