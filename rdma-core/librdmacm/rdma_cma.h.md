# 1. struct rdma_cm_id

## 1.1. 上下文信息

* `struct ibv_context* verbs` 
  * **待查**. 
* `struct rdma_event_channel* channel` 
  * 里面就是一个`int fd` .
* `struct rdma_cm_event* event`
  * **待查**. 
* `void* context` 
  * 用户上下文信息.

## 1.2. Queue Pair

* `struct ibv_qp* qp` 
  * queue pair.
* `enum ibv_qp_type qp_type`
  * qp类型

## 1.3. Completion Queue

* `struct ibv_comp_channel* send_cq_channel`
  * send completion queue channel. **似乎是记录文件描述符的引用计数??**. 
* `struct ibv_cq* send_cq`
  * send completion queue.
* `struct ibv_comp_channel* recv_cq_channel` 
  * 详见`send_cq_channel`.
* `struct ibv_cq* recv_cq` 
  * recv completion queue.

## 1.4. Protection Domain

* `struct ibv_pd* pd`
  * protection domain. 保护内存不被争用, 不被换页.

其他的成员暂不列举.

# 2. 

