---
title: 'std::vector<bool> 中的‘坑’'
date: 2016-11-29 15:10:14
tags:
 - Cpp
categories:
 - 学习小结
feature: /assets/images/features/c_cpp_logo.jpg
---

这两天打算对自己新增的KMC部分进行并行化，其中涉及到对包含有boolean值的vector进行分散和归约操作,但是在使用MPI接口进行编写的时候发现了个问题，当我对`std::vector<bool>`中的元素进行取址的时候编译器傲娇了。

我本意是想把多个进程的bool向量进行逻辑或归约然后分散到所有进程中，这就需要取不同进程中bool序列的地址，大致的代码简化如下:

<!-- more -->

``` Cpp
#include <vector>
#include <iostream>
#include <mpi.h>

int main(int argc, char ** argv)
{
    MPI::Init(argc, argv);
    int rank = MPI::COMM_WORLD.Get_rank();

    std::vector<bool> send(6, false);
    if (rank == 0)
    {
        for (int i = 0; i < 3; ++i)
        {
            send[i] = true;
        }

        std::cout << "Proc 0: " << std::endl;
        for (int i = 0; i < 6; ++i)
        {
            std::cout.width(3);
            std::cout << send[i];
        }
        std::cout << std::endl;
    }
    else if (rank == 1)
    {
        for (int i = 3; i < 6; ++i)
        {
            send[i] = true;
        }
        std::cout << "Proc 1: " << std::endl;
        for (int i = 0; i < 6; ++i)
        {
            std::cout.width(3);
            std::cout << send[i];
        }
        std::cout << std::endl;
    }

    std::vector<bool> recv(6, false);

    MPI_Allreduce(&send[0],
                  &recv[0],
                  6,
                  MPI::BOOL,
                  MPI::LOR,
                  MPI_COMM_WORLD);

    if (rank == 0)
    {
        std::cout << "After all_reduct: " << std::endl;
        for (int i = 0; i < 6; ++i)
        {
            std::cout.width(3);
            std::cout << recv[i];
        }
        std::cout << std::endl;
    }

    MPI_Finalize();
}
```
编译报错说我去了一个临时值的地址：
![](/assets/images/blog_img/2016-11-29-std-vector-bool-中的‘坑’/error.png)

为什么之前的`std::vector<int>`和`std::vector<double>`都不会出现这种问题？

于是还是要求助与万能的谷歌。。。

原来这个问题在Scott Meyers的Effective系列的条例里面都有，他说`std::vector<bool>`并不是一个STL容器，他里面存放的也不是真正的`bool`变量。

在cppreference里面也专门对这个进行了[描述](http://www.cplusplus.com/reference/vector/vector-bool/)：
> Vector of bool
> This is a specialized version of vector, which is used for elements of type bool and optimizes for space.

> It behaves like the unspecialized version of vector, with the following changes:
> - The storage is not necessarily an array of bool values, but the library implementation may optimize storage so that each value is stored in a single bit.
> - Elements are not constructed using the allocator object, but their value is directly set on the proper bit in the internal storage.
> - Member function flip and a new signature for member swap.
> - A special member type, reference, a class that accesses individual bits in the container's internal storage with an interface that emulates a bool reference. Conversely, member type const_reference is a plain bool.
> - The pointer and iterator types used by the container are not necessarily neither pointers nor conforming iterators, although they shall simulate most of their expected behavior.

也就是说为了空间节省，C++标准显式的将`vector<bool>`处理成每个bool值只用一位来存储而不是使用一个字节来存储真正的`bool`值，但是当我们使用`[]`运算符取值的时候，容器会返回一个代理类，这个代理类能够看起来像`bool`值一样被操作，但是对他直接进行取值或者取地址的时候却会出问题，因为他只是个代理对象的临时变量，于是看来我还是尽量不要用`vector`来存储`bool`, 上最原始的数组吧。。。so

``` Cpp
#include <vector>
#include <iostream>
#include <mpi.h>

int main(int argc, char ** argv)
{
    MPI::Init(argc, argv);
    int rank = MPI::COMM_WORLD.Get_rank();

    bool send[6] = {false};
    if (rank == 0)
    {
        for (int i = 0; i < 3; ++i)
        {
            send[i] = true;
        }

        std::cout << "Proc 0: " << std::endl;
        for (int i = 0; i < 6; ++i)
        {
            std::cout.width(3);
            std::cout << send[i];
        }
        std::cout << std::endl;
    }
    else if (rank == 1)
    {
        for (int i = 3; i < 6; ++i)
        {
            send[i] = true;
        }
        std::cout << "Proc 1: " << std::endl;
        for (int i = 0; i < 6; ++i)
        {
            std::cout.width(3);
            std::cout << send[i];
        }
        std::cout << std::endl;
    }

    bool recv[6] = {false};

    MPI_Allreduce(&send[0],
                  &recv[0],
                  6,
                  MPI::BOOL,
                  MPI::LOR,
                  MPI_COMM_WORLD);

    if (rank == 0)
    {
        std::cout << "After all_reduct: " << std::endl;
        for (int i = 0; i < 6; ++i)
        {
            std::cout.width(3);
            std::cout << recv[i];
        }
        std::cout << std::endl;
    }

    MPI_Finalize();
```
这个时候便不会出问题了。

![](/assets/images/blog_img/2016-11-29-std-vector-bool-中的‘坑’/right.png)

