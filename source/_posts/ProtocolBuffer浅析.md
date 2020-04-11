---
title: ProtocolBuffer浅析
comments: true
date: 2020-04-11 20:08:19
categories:
tags:
description:
---
[TOC]
<!--more-->

## ProtocolBuffer浅析

### 背景

ProtocolBuffer是google 定义的一种数据交换的格式，它独立于语言，独立于平台。google 提供了多种语言的实现：java、c#、c++、go 和 python，每一种实现都包含了相应语言的编译器以及库文件。ProtocolBuffer类似于xml、json,不过它更小、更快、也更简单。

#### 与Json对比

目前使用最广泛的数据传输协议为JSON，JSON是一种轻量级的数据交换格式而且层次和结构比较简单和清晰，这里主要对比一下Protocol Buffer和JSON的对比，给出优势和劣势：

**优势**

- 传输数据更小
- 序列化和反序列化更快
- 由于传输的过程中使用的是二进制，没有结构描述文件，无法解析内容，安全性更高

**劣势** 

- 由于传输过程使用的是二进制，自解释性较差，需要原有的结构描述文件才能解析

**实际数据对比**

- 序列化速度：比JSON快20-100倍
- 数据大小：序列化后体积小3倍

## 使用流程

------

Protocol Buffer的使用流程总体可以分为三步，如下图所示：

![image](https://user-gold-cdn.xitu.io/2019/11/9/16e4e03d98a97d18?imageslim)



1. 根据业务创建并定义proto文件
2. 使用Google Protocol Buffer 提供的工具生成对应语言的源文件
3. 将源文件拷贝到工程中，使用Protocol Buffer提供的库序列化或反序列化数据

#### Android中使用

1. 在项目根目录的build.gradle中添加依赖：

   ```java
       dependencies {
           classpath 'com.google.protobuf:protobuf-gradle-plugin:0.8.8'
       }
   ```

2. 在app的gradle中添加插件

   ```groovy
   apply plugin: 'com.google.protobuf'//添加插件
   ```

3. 在app的build.gradle添加如下代码

   ```groovy
   protobuf {
       protoc {
           artifact = 'com.google.protobuf:protoc:3.7.1' // 也可以配置本地编译器路径
       }
   
       generateProtoTasks {
           all().each { task ->
               task.builtins {
                   java {}// 生产java源码
               }
           }
       }
   }
   ```

4. 添加proto文件的路径

   ```groovy
   sourceSets {
       main {
           proto {
               srcDir 'src/main/proto'
           }
           java {
               srcDir 'src/main/java'
           }
       }
   }
   ```

5. 添加protobuf-java和protoc的依赖，其中protoc的依赖很重要，lite版中不需要添加

   ```groovy
   implementation 'com.google.protobuf:protobuf-java:3.7.1'
   implementation 'com.google.protobuf:protoc:3.7.1'
   ```

google推荐在Android项目中使用lite版，lite版本生成的java文件更加轻量，其配置如下：

```groovy
protobuf {
    //配置protoc编译器
    protoc {
        artifact = 'com.google.protobuf:protoc:3.8.0'
    }
    //这里配置生成目录，编译后会在build的目录下生成对应的java文件
    generateProtoTasks {
        all().each { task ->
            task.builtins {
                java {
                    option "lite"
                }
            }
        }
    }
}
dependencies {
  implementation 'com.google.protobuf:protobuf-javalite:3.8.0'
}
```

## 语法解析

------

#### 简单示例

首先创建一个.proto文件，并且在文件中声明如下内容:

```protobuf
syntax = "proto3";		//标明当前proto使用的版本为proto3

option java_package = "com.jon.protocol.protobuf";  //输出的java文件包名
option java_outer_classname = "Test";               //输出的java文件名称

message TestRequest{
    string name = 1;    //名称
    int32 count = 2;
}
```

#### 字段解析

在整个proto文件中数据类型分为基本类型和结构类型，其中结构类型主要为:

- message
- enum
- map

下面分别介绍一下不同结构的作用及规定：

##### message

message表示一个结构，类似于java中类，一个proto文件中可以声明多个message结构：

```protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
 ...
}
```

##### import

message可以引用不同proto文件中的message，只要在proto文件中的最上面声明import即可，如下所示：

```protobuf
import "test.proto";
```

#### enum

enum使用很简单，直接在message中声明enum结构体并且将属性声明为对应的enum即可：

```protobuf
message EnumRequest {
    Corpus corpus = 1;
    enum Corpus {
        UNIVERSAL = 0;
        WEB = 1;
        IMAGES = 2;
        LOCAL = 3;
        NEWS = 4;
        PRODUCTS = 5;
        VIDEO = 6;
    }
}
```

在proto3中，enum第一个值必须为0，主要是为了和基础类型的默认值保持一致

#### map

map是proto3新加的，使用也很简单：

```protobuf
map<key_type, value_type> map_field = N;
//示例如下
message MapRequest {
    map<string, TestRequest> map = 1;
}
```

#### 基础类型

如下

| .proto Type | Notes                                                        | C++ Type | Java Type  | Python Type[2] | Go Type | Ruby Type                      | C# Type    | PHP Type          | Dart Type |
| ----------- | ------------------------------------------------------------ | -------- | ---------- | -------------- | ------- | ------------------------------ | ---------- | ----------------- | --------- |
| double      |                                                              | double   | double     | float          | float64 | Float                          | double     | float             | double    |
| float       |                                                              | float    | float      | float          | float32 | Float                          | float      | float             | double    |
| int32       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead. | int32    | int        | int            | int32   | Fixnum or Bignum (as required) | int        | integer           | int       |
| int64       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead. | int64    | long       | int/long[3]    | int64   | Bignum                         | long       | integer/string[5] | Int64     |
| uint32      | Uses variable-length encoding.                               | uint32   | int[1]     | int/long[3]    | uint32  | Fixnum or Bignum (as required) | uint       | integer           | int       |
| uint64      | Uses variable-length encoding.                               | uint64   | long[1]    | int/long[3]    | uint64  | Bignum                         | ulong      | integer/string[5] | Int64     |
| sint32      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s. | int32    | int        | int            | int32   | Fixnum or Bignum (as required) | int        | integer           | int       |
| sint64      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s. | int64    | long       | int/long[3]    | int64   | Bignum                         | long       | integer/string[5] | Int64     |
| fixed32     | Always four bytes. More efficient than uint32 if values are often greater than 228. | uint32   | int[1]     | int/long[3]    | uint32  | Fixnum or Bignum (as required) | uint       | integer           | int       |
| fixed64     | Always eight bytes. More efficient than uint64 if values are often greater than 256. | uint64   | long[1]    | int/long[3]    | uint64  | Bignum                         | ulong      | integer/string[5] | Int64     |
| sfixed32    | Always four bytes.                                           | int32    | int        | int            | int32   | Fixnum or Bignum (as required) | int        | integer           | int       |
| sfixed64    | Always eight bytes.                                          | int64    | long       | int/long[3]    | int64   | Bignum                         | long       | integer/string[5] | Int64     |
| bool        |                                                              | bool     | boolean    | bool           | bool    | TrueClass/FalseClass           | bool       | boolean           | bool      |
| string      | A string must always contain UTF-8 encoded or 7-bit ASCII text, and cannot be longer than 232. | string   | String     | str/unicode[4] | string  | String (UTF-8)                 | string     | string            | String    |
| bytes       | May contain any arbitrary sequence of bytes no longer than 232. | string   | ByteString | str            | []byte  | String (ASCII-8BIT)            | ByteString | strin             |           |

#### 默认值

- string：空串
- bytes：空字节
- bool：false
- 数字类型：0
- enum：默认值是第一个元素，且值必须为0

#### repeated

repeated修饰的属性类似于jsonArray,也类似于java中的List，该修饰符在格式正确的消息中可以重复任意次（包括0次）

```protobuf
message RepeatRequest {
    repeated TestRequest requests = 1;
}
```

#### 字段编号

1. 字段编号从1开始，不可重复定义
2. 字段编号1-15尽量保持经常访问的字段使用，因为1-15编号在传输的过程中只占用1个字节

#### 字段扩充

日常开发过程中，由于需求的变更，往往需要增加字段，这就涉及到字段的扩充，字段扩充需要达到一个目的：**兼容**

所以Protocol Buffer在字段扩充中定义了如下规则：

1. 不要修改已经存在的字段标号

2. 不用的字段，可以删除，但是编号一定不可以再次使用，建议将字段标为废弃，如加前缀："OBSOLETE_", 或者标记该字段为reserved

   ```protobuf
   reserved 2;
   ```

只要记住上述规则，就能完成字段扩充且老版本也能兼容

### 原理简介

Protocol Buffer 更快更小的主要原因如下：

1. 数据在序列化的时候不会传输字段名，只会传输字段标号，并且没有被设置值的字段是不会序列化和传输
2. 采用可变长度编码，优化数据占用

```protobuf
message TestRequest{
    string name = 1;    //名称
    int32 count = 2;
}
```

上面这个例子中，在序列化时，"name" 、"count"的key值不会参与，由编号1、2代替，这样在反序列化的时候直接通过编号找到对应的key就可以。需要注意的是编号一旦确定就不可以更改，服务端和客户端通过proto通信的时候需要提前定义号数据格式。

- 没有赋值的key，不参与序列化
   序列化时只会对赋值的key进行序列化，没有赋值的不参与，在反序列化的时候直接给默认值即可
- 可变长度编码
   可变长度编码，主要缩减整数占用字节实现，例如java中int占用4个字节，但是大多数情况下，我们使用的数字都比较小，使用1个字节就够了，这就是可变长度编码完成的事
- TLV
   TLV全称为Tag-Length-Value，其中Tag表示后面数据的类型，Length不一定有，根据Tag的值确定，Value就是数据了，TLV表示数据时，减少分隔符的使用，更加紧凑

#### **T-L-V**的数据存储方式

![](https://i.postimg.cc/CKJzqfwJ/a-HR0c-Dov-L3-Vwb-G9h-ZC1pb-WFn-ZXMuamlhbn-Nod-S5pby91c-Gxv-YWRfa-W1h-Z2-Vz-Lzk0.png)

其中Length不一定有，依据Tag确定，例如int类型的数据就只有Tag-Value，string类型的数据就必须是Tag-Length-Value。

#### 数据类型

Protocol Buffer定义了如下的数据类型，其中部分数据类型已经不再使用：

| 类型 | 释义         | 备注                                              |
| ---- | ------------ | ------------------------------------------------- |
| 0    | 可变长度编码 | int32 int64 uint32 uint64 sint32 sint64 bool enum |
| 1    | 64位长度     | fixed64 sfixed64 double                           |
| 2    | value 的长度 | string bytes message packed repeated fiels        |
| 3    | Start Group  | 废弃                                              |
| 4    | End Group    | 废弃                                              |
| 5    | 32位长度     | fixed32 sfixed32 float                            |

#### Tag

上面已经介绍了Protocol Buffer的数据结构及Tag的类型，但是Tag块并不是只表示数据类型，其中数据编号也在Tag块中，Tag的生成规则如下：

```protobuf
(field_number << 3) | wire_type
```

其中Tag块的后3位表示数据类型，其他位表示数据编号

### 可变长度编码

Java中整数类型的长度都是确定的，如int类型的长度为4个字节，可表示的整数范围为-2^31——2^31-1，但是实际开发中用到的数字均比较小，会造成字节浪费，可变长度编码就能很好的解决这个问题，可变长度编码规则如下：

- 字节最高位表示数据是否结束，如果最高位为1，则表示后面的字节也是该数据的一部分

举个例子：

![image](https://i.postimg.cc/BvtX2nt0/16e69876dd1d471a-imageslim.png)

其中第一个字节由于最高位为1，则后面的字节也是前面的数据的一部分，第二个字节最高位为0，则表示数据计算终止，由于Protocol Buffer是低位在前，整体的转换过程如下：

![image](https://i.postimg.cc/hjWchDdM/16e6997eb773d308-imageslim.png)



10000001 00000011 ——> 00000110000001 表示的10进制数为：2^0 + 2^7 + 2^8 = 385
 通过上面的例子可以知道一个字节表示的数的范围0-128，上面介绍的Tag生成算法中由于后3位表示数据类型，所以Tag中1-15编号只占用1个字节，所以确保编号中1-15为常用的，减少数据大小。

可变长度编码唯一的缺点就是当数很大的时候int32需要占用5个字节，但是从统计学角度来说，一般不会有这么大的数.

### 案例分析

上面介绍了Protocol Buffer的原理，现在通过实例来展示分析过程，我们定义的proto文件如下：

```protobuf
message TestRequest{
    string name = 1;    //名称
    int32 count = 2;
}
```

其序列化后的字节数据如下：

![testproto.png](https://i.postimg.cc/QCpV0ttL/testproto.png)

前面介绍过Protocol Buffer的**数据结构为TLV，其中L不是必须的，根据T的类型来确定**
先看下第一个字节：

![image1](https://i.postimg.cc/BQs5H23f/16e69c909d30c944-imageslim.png)

这里字节最高位为0，所以该Tag就用这一个字节表示，其中后3位表示类型，前面表示字段编号，所以:

这里字节最高位为0，所以该Tag就用这一个字节表示，其中后3位表示类型，前面表示字段编号，所以:
 file_num = 0001 = 1
 type = 010 = 2
 上面介绍过type=2，则后面有Length，按照可变长度编码规则，知道表示长度的字节为：

![image2](https://i.postimg.cc/WbC2xw1Q/16e69cd62541fa3a-imageslim.png)

所以Length=4,则value的长度是4个字节，直接取出后面4个字节：

![image3](https://i.postimg.cc/ncyJ3Vdx/16e69cf670400a24-imageslim.png)

这4个字节对应的就是test
 再看下一组：

![image4](https://i.postimg.cc/zfp4YqkV/16e69d19847ddfcf-imageslim.png)

由上面的Tag知道： file_num=2  type=0
 前面介绍过type=0，后面没有Length，直接就是value，

![image5](https://i.postimg.cc/PfLfy7tm/16e69d3c5893f001-imageslim.png)

value=1，通过上面的解析可以知道

1. file_num=1 value=test
2. file_num=2 value=1
    这样解析就结束了

上面介绍了Protocol Buffer的原理，解释了为什么Protocol Buffer更快，更小，这里再总结一下：

1. 序列化的时候，不序列化key的name，只序列化key的编号
2. 序列化的时候，没有赋值的key，不参与序列化，反序列化的时候直接使用默认值填充
3. 可变长度编码，减小字节占用
4. TLV编码，去除没有的符号，使数据更加紧凑





## 长链接相关浅析

### 定义

长连接，指在一个连接上可以连续发送多个[数据包](https://baike.baidu.com/item/数据包/489739)，在连接保持期间，如果没有数据包发送，需要双方发链路检测包。

### http长链接

HTTP协议本身是应用层协议，其长连接和短连接本质上是TCP长连接和短连接。 HTTP1.1规定了默认保持长连接（HTTP persistent connection ，也有翻译为持久连接），数据传输完成了保持TCP连接不断开（不发RST包、不四次握手），等待在同域名下继续用这个通道传输数据。

### http协议里的keep-alive

http1.1 协议里增加了 keepalive的支持， 并且默认开启。

客户端和服务端在建立连接并完成request后并不会立即断开TCP连接，而是在下次request来临时复用这次TCP连接。但是这里也必须要有TCP连接的timeout时间限制。不然会造成服务端端口被长期占用释放不了。

![keepalive示意](https://upload-images.jianshu.io/upload_images/1967392-8335feea68acc6a2.png)

客户端和服务端在建立连接并完成request后并不会立即断开TCP连接，而是在下次request来临时复用这次TCP连接。但是这里也必须要有TCP连接的timeout时间限制。不然会造成服务端端口被长期占用释放不了。

对于不适用keepalive的request来说，不管是客户端还是服务端都是通过TCP的链接的断开知道request的结束（TCP 挥手时会check 数据包的 seq， 保证数据完整性）。
 支持keepalive后，如何知道request结束了呢？

1. 在Http1.1的版本里， 解决方案是request 和reponse里使用**contentLength**来帮助确认是否收到全部数据。
2. 动态生成的文件没有Content-Length，服务器是不可能预先知道内容大小，这时就可以使用Transfer-Encoding：chunk模式来传输数据了,即如果要一边产生数据，一边发给客户端，服务器就需要使用"Transfer-Encoding: chunked"这样的方式来代替Content-Length。这时候就要根据chunked编码来判断，chunked编码的数据在最后有一个**长度为0**chunked块，表明本次传输数据结束.

#### **管道机制(Pipelining)**

![](https://i.postimg.cc/3wgmkwqx/1920px-HTTP-pipelining.png)

HTTP Pipelining是把多个HTTP请求放到一个TCP连接中一一发送，而在发送过程中不需要等待服务器对前一个请求的响应；只不过，客户端还是要按照发送请求的顺序来接收响应。但就像在超市收银台或者银行柜台排队时一样，你并不知道前面的顾客是干脆利索的还是会跟收银员/柜员磨蹭到世界末日。不管怎么说，服务器（即收银员/柜员）是要按照顺序处理请求的，如果前一个请求非常耗时（顾客磨蹭），那么后续请求都会受到影响，这就是所谓的线头阻塞（Head of line blocking）。

HTTP2.0 新增了多路复用的技术，通过在应用层http协议和传输层tcp协议之间增加一个二进制分帧层，将请求分为不同的帧，每一帧会标识出该帧属于哪个流，流是多个帧组成的数据流。所谓多路复用，即在一个TCP连接中存在多个流，即可以同时发送多个请求，对端可以通过帧中的表示知道该帧属于哪个请求。在客户端，这些帧乱序发送，到对端后再根据每个帧首部的流标识符重新组装。通过该技术，可以避免HTTP旧版本的队头阻塞问题，极大提高传输性能。



### WebSocket

WebSocket是HTML5新增的协议，它的目的是在浏览器和服务器之间建立一个不受限的双向通信的通道，比如说，服务器可以在任意时刻发送消息给浏览器。

为什么传统的HTTP协议不能做到WebSocket实现的功能？这是因为HTTP协议是一个请求－响应协议，请求必须先由浏览器发给服务器，服务器才能响应这个请求，再把数据发送给浏览器。换句话说，浏览器不主动请求，服务器是没法主动发数据给浏览器的。

这样一来，要在浏览器中搞一个实时聊天，在线炒股（不鼓励），或者在线多人游戏的话就没法实现了，只能借助Flash这些插件。

也有人说，HTTP协议其实也能实现啊，比如用轮询或者Comet。轮询是指浏览器通过JavaScript启动一个定时器，然后以固定的间隔给服务器发请求，询问服务器有没有新消息。这个机制的缺点一是实时性不够，二是频繁的请求会给服务器带来极大的压力。

Comet本质上也是轮询，但是在没有消息的情况下，服务器先拖一段时间，等到有消息了再回复。这个机制暂时地解决了实时性问题，但是它带来了新的问题：以多线程模式运行的服务器会让大部分线程大部分时间都处于挂起状态，极大地浪费服务器资源。另外，一个HTTP连接在长时间没有数据传输的情况下，链路上的任何一个网关都可能关闭这个连接，而网关是我们不可控的，这就要求Comet连接必须定期发一些ping数据表示连接“正常工作”。

以上两种机制都治标不治本，所以，HTML5推出了WebSocket标准，让浏览器和服务器之间可以建立无限制的全双工通信，任何一方都可以主动发消息给对方。

### WebSocket协议

WebSocket并不是全新的协议，而是利用了HTTP协议来建立连接。我们来看看WebSocket连接是如何创建的。

首先，WebSocket连接必须由浏览器发起，因为请求协议是一个标准的HTTP请求，格式如下：

```http
GET ws://localhost:3000/ws/chat HTTP/1.1
Host: localhost
Upgrade: websocket
Connection: Upgrade
Origin: http://localhost:3000
Sec-WebSocket-Key: client-random-string
Sec-WebSocket-Version: 13
```

该请求和普通的HTTP请求有几点不同：

1. GET请求的地址不是类似`/path/`，而是以`ws://`开头的地址；
2. 请求头`Upgrade: websocket`和`Connection: Upgrade`表示这个连接将要被转换为WebSocket连接；
3. `Sec-WebSocket-Key`是用于标识这个连接，并非用于加密数据；
4. `Sec-WebSocket-Version`指定了WebSocket的协议版本。

随后，服务器如果接受该请求，就会返回如下响应：

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: server-random-string
```

该响应代码`101`表示本次连接的HTTP协议即将被更改，更改后的协议就是`Upgrade: websocket`指定的WebSocket协议。

版本号和子协议规定了双方能理解的数据格式，以及是否支持压缩等等。如果仅使用WebSocket的API，就不需要关心这些。

现在，一个WebSocket连接就建立成功，浏览器和服务器就可以随时主动发送消息给对方。消息有两种，一种是文本，一种是二进制数据。通常，我们可以发送JSON格式的文本，这样，在浏览器处理起来就十分容易。

为什么WebSocket连接可以实现全双工通信而HTTP连接不行呢？实际上HTTP协议是建立在TCP协议之上的，TCP协议本身就实现了全双工通信，但是HTTP协议的请求－应答机制限制了全双工通信。WebSocket连接建立以后，其实只是简单规定了一下：接下来，咱们通信就不使用HTTP协议了，直接互相发数据吧。

安全的WebSocket连接机制和HTTPS类似。首先，浏览器用`wss://xxx`创建WebSocket连接时，会先通过HTTPS创建安全的连接，然后，该HTTPS连接升级为WebSocket连接，底层通信走的仍然是安全的SSL/TLS协议。



参考资料：

proto3官网指南：https://developers.google.com/protocol-buffers/docs/proto3

protobuf-gradle-plugin：https://github.com/google/protobuf-gradle-plugin

图床：https://postimages.org/

博客：https://juejin.im/post/5dcbf630e51d451bfe5bb21b

廖雪峰websocket：https://www.liaoxuefeng.com/wiki/1022910821149312/1103303693824096