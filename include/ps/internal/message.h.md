# 概要

头文件message.h定义了Message的结构. 

* 首先使用了自定义的`SArray`, i.e. Smart Array. 共享数据, 减少数据拷贝, 且提供了类似vector的接口.
* 元数据`Meta`使用了Protobuf, 进行了数据压缩.
* 消息分层比较清晰. `Node`包含节点的角色、id、ip、端口信息; `Control`包含了命令信息、签名等; `Meta`是元数据, 包含时间戳、发送者、接受者、控制信息等; `Message`才是发送的信息, 包含元数据和发送的数据. 
* 参数有key-value组成, 对应`KVPairs`.

# struct Node

这个结构体维护节点的属性, 如

* id
* 端口号
* 角色
* 主机名

等等.

# struct Control

控制信息结构体.

* `node` node列表 **??? 干什么用的 ???** 
* `cmd` 控制命令 { EMPTY, TERMINATE, ADD_NODE, BARRIER, ACK, HEARTBEAT}
  * 默认为EMPTY
* `barrier_group` barrier group的id
* `msg_sig` 消息签名

# struct Meta

元信息结构体.

* `head` 头部, **干什么用的?** 
* `customer_id` 详见customer.h.md.
* `sender`和`recver`. 发送者和接收者的node id.
* `vector<DataType> data_type` 和Message的data长度相同. 是data每一项的对应类型.
* `control` 消息携带的控制信息.



# struct Message

* `meta` 消息携带的元信息.
* `vector<SArray<char>> data` 消息携带的数据.

