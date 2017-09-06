# 概述

看这个代码需要随时查阅[ZeroMQ API](http://api.zeromq.org/).

另外, zmq的api地址很有规律, 只要输入`http://api.zeromq.org/4-2:符号名`就能快速查找zmq api. 注意符号名的下划线要替换成减号.

# C++11速查

[C++11新特性:Lambda函数(匿名函数) - DoubleLi - 博客园](http://www.cnblogs.com/lidabo/p/3908663.html)

# context_

是zmq socket工作的上下文, 用来确定socket的类型. 可以被多个线程共享.

该成员在`ZMQVan::Start()`中通过调用[`zmq_ctx_new()`](http://api.zeromq.org/4-2:zmq-ctx-new)来初始化, 接下来通过[`zmq_ctx_set()`](http://api.zeromq.org/4-2:zmq-ctx-set)进行一些设置.

该成员在`ZMQVan::Stop()`中通过调用`zmq_ctx_destroy()` 来销毁.

# Bind和receiver_

```cpp
int Bind(const Node& node, int max_retry) override;
```

返回最终绑定的端口号(有可能和`node.port`不一样). 如果绑定失败, 返回-1.

## 1) 创建server socket

首先, 调用[`zmq_socket`](http://api.zeromq.org/4-2:zmq-socket) 来创建一个ZMQ_ROUTER类型的socket. 

由于zmq_socket的API文档的排版辨识度不高, 这里简单整理一下原文档的纲要:

原文档主要是在讲, zmq socket有如下几种通讯模式 (比较生僻的就不列举了):

* Client-server pattern
  * ZMQ_CLIENT和ZMQ_SERVER
* Publish-subscribe pattern
  * ZMQ_PUB和ZMQ_SUB
  * ZMQ_XPUB和ZMQ_XSUB
* Request-reply Pattern
  * ZMQ_REQ和ZMQ_REP
  * ZMQ_DEALER和ZMQ_ROUTER

文档还说, **Requiest-reply Pattern指的是一个request/dealer客户端向多个reply/router服务器发送请求, 并分别获取响应的通讯模式**.

## 2) 绑定端口

调用[`zmq_bind`](http://api.zeromq.org/4-2:zmq-bind)绑定端口. 首先尝试绑定到`node.port`, 如果绑定失败, 尝试绑定随机端口, 直到最大重试次数max_retry.

# RecvMsg

```cpp
int RecvMsg(Message* msg) override;
```

返回收到的字节数.

该函数是纯虚函数`Van::RecvMsg`的具体实现. 负责在`Van::RecvMsg`线程中接收消息数据. 具体过程如下

## 1) 收取数据

该函数调用[`zmq_msg_recv`](http://api.zeromq.org/4-2:zmq-msg-recv)收取数据. 接下来调用[`zmq_msg_data`](http://api.zeromq.org/4-2:zmq-msg-data)获得zmsg底层缓冲区的首地址.

该函数采用**阻塞**式I/O的方式来调用zmq.

## 2) 解析数据

为了方便解析message, ps-lite在使用zmq的时候约定收发方把ps-lite message拆成三个`zmq_msg`. 依次为sender/receiver, 其他meta信息, 以及数据负载.

# Connect和senders_

```cpp
void Connect(const Node& node) override
```

`senders_` 是一个节点ID到zmq socket对象的映射. `senders_` 中的zmq socket是ZMQ_DEALER类型, 和`receiver_` 的ZMQ_ROUTER相对应.

首先检查要连接的node id是否已经存在socket, 如果有, 就关闭并释放这个socket. 

之后用`zmq_socket`创建一个DEALER类型的socket.

接下来用[`zmq_setsockopt`](http://api.zeromq.org/4-2:zmq_setsockopt)设置socket的选项. 并尝试用[`zmq_connect`](http://api.zeromq.org/4-2:zmq_connect)来连接到对方节点的server socket.

连接成功之后, 把节点id和socket存入senders_.

# SendMsg

```
int SendMsg(const Message& msg) override
```

返回发送字节数.

先把meta data封装成一个zmq message发送出去. 再把负载中的每一个`SArray<char>`发送出去. 



