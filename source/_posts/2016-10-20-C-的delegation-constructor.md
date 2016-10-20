---
title: C++的delegation constructor
date: 2016-10-20 09:03:48
tags:
 - Cpp
 - delegation constructor
categories:
 - 学习小结
feature: /assets/images/features/c_cpp_logo.jpg
---
昨晚给KMCLibX的Process类添加了一个新的构造函数，目的是想快速添加slow_flag到process中。结果我想能不能在一个构造函数中调用同类中的另一个构造函数，于是我发现了C++11有了个新特性叫做[delegation constructor](https://en.wikipedia.org/wiki/C%2B%2B11#Object_construction_improvement)

在C++03中是不允许在一个构造函数中调用另一个构造函数的，要实现这种效果需要写一个共有的初始化函数然后再不同的构造函数中调用，例如：
<!-- more -->

``` Cpp
class Foo
{
public:
    Foo(char x);
    Foo(char x, int y);
    ...
private:
    void init(char x, int y);
};

Foo::Foo(char x)
{
    init(x, int(x) + 7);
    ...
}

Foo::Foo(char x, int y)
{
    init(x, y);
    ...
}

void Foo::init(char x, int y)
{
    ...
}
```

C++11允许了我们直接在一个构造函数中直接调用另一个构造函数。语法如下：
``` Cpp
class SomeType
{
    int number;

public:
    SomeType(int new_number) : number(new_number) {}
    SomeType() : SomeType(42) {}
};
```

注意并不是说直接可以再函数体中调用，而是要在**初始化列表**中调用。

现在的Process多态构造函数就象这样了：
``` Cpp
Process::Process(const Configuration & first,
                 const Configuration & second,
                 const double rate,
                 const std::vector<int> & basis_sites,
                 const bool slow) :
    Process(first, second, rate, basis_sites, {}, {}, -1, {}, slow)
{
    // NOTHING HERE.
}
```

