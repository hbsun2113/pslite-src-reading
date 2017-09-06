# Start

```cpp
void Van::Start()
```

要想移植RDMA, 首先要了解ps-lite的Van是怎么启动的.

## 1. 验明正身

Start()首先查询环境变量, 并询问`PostOffice`来获知自己是不是scheduler.

用到的环境变量有:

```sh
DMLC_PS_ROOT_URI
DMLC_PS_ROOT_PORT
```

## 2. 获取IP和端口号

也是从环境变量中获取IP和端口号. 用到的环境变量有:

```
DMLC_NODE_HOST
DMLC_INTERFACE # 如果前者为空
PORT
```

## 3. 绑定并监听端口

调用`ZMQVan::Bind(my_node_, is_scheduler?0:40)`来绑定和监听端口, 并记录返回的端口号. 这里, 第二个参数的含义是重试次数.

> 随想: 在RdmaVan中override Van::Bind

## 4. 连接到scheduler

调用`ZMQVan::Connect(scheduler_)`连接到调度者.

> 随想: 在RdmaVan中override Van::Connect

## 5. 开启接收者线程

开启一个新的线程来执行`Van::Receiving`, 并以`unique_ptr`的形式把线程句柄记录在`receiver_thread_`中.

然后向scheduler发送一条`Control::ADD_NODE`类型的控制消息.

之后还要开启Resender. 目前还搞不懂Resender的具体原理.

## 6. 开启心跳线程

开启一个新的线程来执行`Van::HeartBeat`, 并以unique_ptr的形式把线程句柄记录在`heartbeat_thread_`中.

# Receiving

```cpp
void Van::Receiving()
```

van.h给出的注释是

> thread function for receiving

在Van::Start()中创建Receiving线程.

# PackMeta和UnpackMeta

用Protobuf来序列化\反序列化Meta数据.

