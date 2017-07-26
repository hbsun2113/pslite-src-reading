# 概述

SArray意为Shared Array. 它和`std::vector`很像, 比如它有data(), size(), operator[], resize(), clear()等函数.

# 比赛中可能用到?

目前看来, 发送SArray只会用到data()和size()函数, 而接收SArray要用到SArray的构造方法.

# 底层原理

## 数据结构

首先抛开shared_ptr, 单纯看数据结构, 它就是一个简单的vector实现. 这个vector用`size_`和`capacity_`来维护vector的逻辑空间和物理空间:

* 当`size_`大于`capacity_`时, 就给`capacity_`加5, 
* 用新的`capacity_`申请 (**`new[]`**)新内存, 
* 调用memcpy拷贝原始内容到新内存, 
* 释放旧内存 ( **`delete[]`** ). 

## shared_ptr

然后再看`shared_ptr`. `shared_ptr`只是封装C指针, 为其增添了引用计数的功能. 在SArray中只调用了`shared_ptr`的两个函数: get和reset.

### get

这个用来获取`shared_ptr`所包装的C指针. 在SArray中, 这个C指针只可能是`new[]` 出来的动态数组的首地址, 因此对其进行下标访问是有意义的.

###  reset

这个用来替换shared_ptr的底层C指针. 根据[STL参考手册](http://en.cppreference.com/w/cpp/memory/shared_ptr/reset), reset的功能如下:

> Replaces the managed object with an object pointed to by `ptr`. Optional deleter `d` can be supplied, which is later used to destroy the new object when no `shared_ptr` objects own it. By default, [`delete`](http://en.cppreference.com/w/cpp/language/delete) expression is used as deleter. Proper [`delete`](http://en.cppreference.com/w/cpp/language/delete) expression corresponding to the supplied type is always selected, this is the reason why the function is implemented as template using a separate parameter `Y`.

SArray给参数d传递的实参是如下的lambda表达式:

```c++
[](V* data){ delete[] data; }
```

lambda表达式可以认为是简化的函数指针, 小括号里面是形参, 大括号里面是函数体.