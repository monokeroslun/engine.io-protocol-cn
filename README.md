# engine.io-protocol-cn

## 1 译者序

这是中文版的engine io协议。原协议见：https://github.com/Automattic/engine.io-protocol。

## 2 前言

这个文档描述了Engine.IO 协议。该协议的实现，见 engine.io-parser,engine.io-client和 engine.io。

## 3 修订

这是Engine.IO的第三个修订版本。

## 4 Engine.IO会话（session）的分析

1 传输建立了一个指向Engine.IO URL的连接。

2 服务器端返回了一个使用json格式的，意图为打开传输连接数据包（原文为open packet:
    
    1 sid ：会话 id(String类型)
    
    2 upgrades ：可能的传输升级。（字符串类型的数组）
    
    3 pingTimeout : 服务器端配置 ping 超时，用于客户端探查
    服务器端有无响应。（Number类型）

3 当客户端收到服务器按照一定的周期发送的 ping 包时，需要发送 pong包作为响应。

4 客户端和服务器可以任意交换 message 包。

5 轮询传输会发送一个 close包，因为它会不停的打开，关闭。（Polling transports can send a close packet to close the socket, since they're expected to be "opening" and "closing" all the time.）

![image](img/1.png)

# URLs

  一个 Engine.IO的url由以下部分组成：
  
  
 /engine.io/[?]
 
 
 - engine.io的路径原则上只能被在它之上的高层次的框架（该框架的协议承载层为engine.io 协议）改变。
 
 
 - [?] 询问部分是可选择的，有四种可供选择的关键字：
 
 
     - `transport`: 暗示了传输类型的名称。支持的属性值有：polling,flashsocket,websocket。
     
     - `j`:如果传输类型是`polling`，并且需要JSONP响应，j必须设置为JSONP响应的索引。(j: if the transport is polling but a JSONP response is required, j must be set with the JSONP response index.)
     
     -`sid`:如果服务器曾经给予客户端一个会话id,那么这个id一定会包含在询问部分（[?]）
     
     -`b64`:如果客户端不能支持XHR2，`b64=1`将会被通过[?]部分发送给服务器，用于告知服务所有二进制数据要是用base64编码。
    ( xhr2的内容见 http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)
    
FAQ: `engine.io`部分是可更改的么？

可以的。服务器端可以在不同的路径片段下去拦截请求。

FAQ: 什么决定了URL中要包含的属性？

一般地，url中仅仅是表明该请求是否由Engine.IO实例来处理，所以仅仅需要engine.io(前缀)和资源。（
It's convention that the path segments remain only that which allows to disambiguate whether a request should be handled by a given Engine.IO server instance or not. As it stands, it's only the Engine.IO prefix (/engine.io) and the resource (default by default).）

## Encoding

这里有两种不同的编码格式：

1 packet
2 payload

## Packet

一个编码过的包（packet）可以是UTF-8字符串或者是二进制数据。这个（packet）作用于字符串的数据包编码格式要遵循：

`<packet type id > [<data>] `

例如：

`2probe`


二进制的数据编码也是相同的。当发送二进制数据时，数据包类型id（packet type id）将会在数据中的第一个字节中出现。之后就是真正要传送的数据。

`4|0|1|2|3|4|5`

在上述的例子中每一个字节都会用一个pipe符号来分割并用一个整数来表示。上述的例子是一个消息（message）类型的数据包，并且包含对应于整数值为0，1，2，3，4，5的二进制数据。

数据包类型id(packet type id)是一个整数。以下是可以接受的数据包类型。

#### 0 open

当一个新的传输连接打开时，服务器端会发送该类型数据包（用于确认）

#### 1 close

请求关闭这条传输连接，但是并不会自动关闭连接。

#### 2 ping

用于客户端发送。 服务器端需使用包含同样数据的pong数据包作为应答。

1 客户端发送： 2probe

2 服务器端发送：3 probe

#### 3 pong

用于服务器端发送，应答客户端发送的ping数据包。

#### 4 message

实际发送的消息，客户端和服务器端都可以通过其回调函数得到数据。

**例1**

1 服务器端发送： 4HelloWorld

2 客户端收到并调用其回调函数`socket.on('message,function(data){console.log(data);});`

**例2**

1 客户端发送：4HelloWorld

2 服务器端接收到并调用其回调函数`socket.on('message',function(data){console.log(data);});`

#### 5 upgrade

在engine.io切换另一个传输连接之前，需要进行测试是否服务器端和客户端可以在这个传输连接上进行通信。如果测试成功，客户端会发送一个upgrade数据包用于请求刷新它在旧连接上的缓存并且切换到新的传输连接上去。

#### 6 noop

一种空操作数据包（noop packet）.主要用于强制一个轮询周期当一个新的websocket连接被接收。

**例**

1 客户端通过新的传输进行连接。

2 客户端发送2probe.

3 服务器端需要接收并相应3probe.

4 客户端接收并发送5.

5 服务器端刷新并关闭旧的连接并切换到新的连接。



    
   
     














