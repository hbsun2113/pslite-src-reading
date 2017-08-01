# 概述

这个头文件里定义了

* `KVPairs`: key-values (注意不仅仅是value. 一个key可以对应索引一组value) 列表. 必须按照键升序排列.
* `KVMeta`: kv请求的元信息.
* `KVWorker`: 是SimpleApp的派生类, 用来向`KVServer` Push/Pull 对.
* `KVServer`: 是SimpleApp的派生类，用来保存key-values数据.

# KVPairs

一个简单的结构体. 按照键升序排列, 封装key-values list.

* `keys`: 按照升序排列的键. key可以是uint32或uint64类型.
* `vals`: 相应的值列表.
* `lens`: 每个值的长度的列表. len为int类型. 关于长度的计算这里不赘述, 详见kv_app.h中的注释.

注意: KVPairs的**value必须是primitive-type**.

# KVMeta

每次push/pull操作所需的元信息.

* `cmd`: 不太懂.

  > 原话: An optional command sent to the servers.

* `push`: 是否为push.

* `sender`: 发送者的node id.

* `timestamp`: 操作的时间戳

# KVServer

很重要的类. 用来保存key-values数据, 同时定义push和pull的行为.

## C++11速查

```
/* 把父类成员当成自己成员来用, 避免用父类名称来引用, 让代码更简洁 */
using SimpleApp::obj_;

/* 类似于C的typedef. 
   定义一个函数对象类型, 封装一个没有参数列表, 返回void的函数 */
using Callback = std::function<void()>; 
```

