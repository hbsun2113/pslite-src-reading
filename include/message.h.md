# 概要

头文件message.h定义了Message的结构. 

Message实际上是个四层嵌套结构: 每个Message含一个Meta, 每个Meta含一个Control, 每个Control含一组Node.

* Message寄放消息的实际内容(类型为SArray<class T>). 
* Meta记录消息的收发方, 消息类型, 是push还是pull, 数据类型等.

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
* `data_type` 和Message的data长度相同. 是data每一项的对应类型.
* `control` 消息携带的控制信息.



# struct Message

* `meta` 消息携带的元信息.
* `data` 消息携带的数据.

