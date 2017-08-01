# 概述

Customer是用来通信的对象. **每个连接对应一个Customer对象.**[ [ps-lite源码分析] ](http://www.voidcn.com/blog/kangroger/article/p-6643933.html)

* 当Customer**作为发送方时, 它跟踪每个所发出的请求的响应**.
* 同时Customer还开了一个接收进程, 用来处理`msg.meta.customer_id`等于自己的`customer_id`的消息.

# 重要函数

## WaitRequest()

等待请求完成.

## Accept()

接受一个从Van收到的message.

## Receiving()

Customer的接收进程.