# 1. 资料导航

## 1.1. MXNet和KVStore

### 1.1.1. 源码编译

详见 [如何从源码编译安装MXNet的python模块?](Build_MXNet_from_source.md)

### 1.1.2. MXNet和KVStore架构理解

* Symbol: 声明式 (Declarative) 的符号表达式 (就是更贴近数学语言的程序语言啦)
* NDArray: 命令式 (Imperative) 的张量计算 (一脸懵逼???什么是张量? 就是有现实意义、有计量单位的数组啦. [链接: 数学不行还学AI-第四话-知乎](https://zhuanlan.zhihu.com/p/25268020) )
* **KVStore: ** 多设备间的数据交互. 通过push/pull实现交互.
  * `push`: worker向server推键值对
  * `pull`: server从worker按照键来取值
  * `updater`: push同一个键时所作的操作. 默认为覆盖. 用户可以绑定自己的updater回调函数, 来实现梯度下降\权值更新等目的.

### 1.1.3 参考文献

* [[EB/OL] MXNet设计和实现简介](https://github.com/dmlc/mxnet/issues/797)
* [[J] A Flexible and Efficient Machine Learning Library for Heterogeneous Distributed Systems](https://www.cs.cmu.edu/~muli/file/mxnet-learning-sys.pdf)
* [[EB/OL] MXNet System Architecture](http://mxnet.io/architecture/overview.html)
* [[EB/OL] Deep Learning Programming Style](http://mxnet.io/architecture/program_model.html)

## 1.2. pslite架构理解

### 1.2.1 角色

三种角色: server\worker\scheduler

![pslite架构](https://raw.githubusercontent.com/dmlc/dmlc.github.io/master/img/ps-arch.png)

- **Worker**. A worker node performs the main computations such as reading the data and computing the gradient. It communicates with the server nodes via `push` and `pull`. **For example, it pushes the computed gradient to the servers, or pulls the recent model from them.**
- **Server.** A server node **maintains and updates the model weights**. Each node maintains only a part of the model.
- **Scheduler.** The scheduler node monitors the aliveness of other nodes. It can be also used to send control signals to other nodes and collect their progress.

### 1.2.2. 代码结构

`2017-7-24 更新:` 请参照[明矾队长的工作笔记](http://222.195.93.137/gitlab/lmf/grpc_rdma/blob/master/%E5%B7%A5%E4%BD%9C%E8%AE%B0%E5%BD%95.docx) .

![ps-lite代码结构](http://img.voidcn.com/vcimg/000/006/461/913_c87_27e.svg)

### 1.2.3. 参考文献

1. [[EB/OL]PS-Lite Documents](http://ps-lite.readthedocs.io/en/latest/). 这个文档介绍了PS-Lite的原理和用法.
   * 追评: 感觉这个文档没什么用. 除了介绍了几个**ps-lite要用的环境变量**, 其他的都还没写完.
2. [[EB/OL]PS-Lite源码分析 - 程序园](http://www.voidcn.com/blog/kangroger/article/p-6643933.html). 这个博文详细分析了PS-Lite的结构, 各个模块(类)的职能.
   * 追评: 很详细. 语言组织不够流畅. 但是仔细看很有收获.
3. [[EB/OL]PS-Lite源码剖析 - 作业部落](https://www.zybuluo.com/Dounm/note/529299). 这个博文详细剖析了PS-Lite的类职能, 消息处理流程和实现细节. 并给出了简单的例子.
4. [[EB/OL]【深度学习&分布式】Parameter Server详解 - 仙道菜 - CSDN博客](http://blog.csdn.net/cyh_24/article/details/50545780). 这个博文详细解释了Parameter Server的优点(重点在几个动图), 架构(重点在push, pull, range push和range pull), 以及build blocks(由哪些算法和方法组成).
5. [[J] Scaling Distributed Machine Learning with the Parameter Server](https://www.cs.cmu.edu/~muli/file/parameter_server_osdi14.pdf). 注意看Algorithm 1, 里面有ps-lite如何用来做分布式的随机梯度下降.
6. [明矾队长的工作笔记](http://222.195.93.137/gitlab/lmf/grpc_rdma/blob/master/%E5%B7%A5%E4%BD%9C%E8%AE%B0%E5%BD%95.docx) .

## 1.3. RDMA & Infiniband

### 1.3.1. RDMA编译相关

需要链接这些库:

```
-lrdmacm -libverbs
```

### 1.3.2. 参考文献

1. [Building an RDMA-Capable Application with IB Verbs](http://www.hpcadvisorycouncil.com/pdf/building-an-rdma-capable-application-with-ib-verbs.pdf)
2. [RDMA Read and Write with IB Verbs](http://www.hpcadvisorycouncil.com/pdf/rdma-read-and-write-with-ib-verbs.pdf)
3. [Basic Flow Control for RDMA Transfers](http://www.hpcadvisorycouncil.com/pdf/vendor_content/basic-flow-control-for-rdma-transfers.pdf)
4. [RDMA Aware Programming User Manual](http://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf) 这可以说是RDMA的官方API文档
5. [RDMA mojo : blog on RDMA technology and programming](http://www.rdmamojo.com/) RDMA创建者之一的博客
6. [RDMA和IB的源码](https://github.com/linux-rdma/rdma-core/). RDMA和IB实际上是Linux源码的一部分.

### 1.3.3. 参考代码

1. [Basic Flow Control for RDMA Transfers的代码](https://sites.google.com/a/bedeir.com/home/rdma-file-transfer.tar.gz). 在appendix目录中也有本地备份. 

# 2. 经验总结

## 2.1. 调试技巧

### 2.1.1. 多进程调试

1. 在每个进程的main添加睡眠时间, 使得你有足够的时间打断点.
2. 找到pid, 以便下一步把gdb附加到进程(Attach to process)
   * 一般来说可以用`ps -al | grep 用户名` 的方式找到要调试的进程的pid.
   * 还可以用htop. htop类似于windows的任务管理器, 功能更强大.
3. 用`gdb -p pid` 开始调试进程.
4. 在`(gdb)`命令提示符中, 用`source 文本文件`加载调试脚本, 避免重复输入打断点的命令, 可以让打断点更从容一些.
5. **找到每个进程的角色:** scheduler会打印一些调试信息. 这些信息会显示出每个角色分别在**哪个node的哪个端口**监听着. 用`netstat -antup`找到占用端口的pid.

### 2.1.2. 调试时一定要知道自己到哪了

常见的命令有backtrace(bt) - 显示调用堆栈, list-显示当前及周围几行的代码.

### 2.1.3. 打印中间变量的值

print, display