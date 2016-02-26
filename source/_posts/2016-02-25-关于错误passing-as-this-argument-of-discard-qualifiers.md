---
title: "关于错误passing '...' as 'this' argument of '...' discard qualifiers"
date: 2016-02-25 22:07:12
tags:
  - Cpp
categories:
  - 学习小结
---
今天又碰到了个新问题，先上代码：
vect.cpp
``` Cpp
...
// private methods
    // calculates magnitude from x and y
    double Vector::get_mag(double a, double b)
    {
        return sqrt(a * a + b * b);
    }

    double Vector::get_ang(double a, double b)
    {
        if (a == 0.0 && b == 0.0)
            return 0.0;
        else
            return atan2(b, a);
    }
...
```
<!-- more -->
vect.h
``` Cpp
// vect.h -- Vector class with <<, mode state
#ifndef VECTOR_H_
#define VECTOR_H_
#include <iostream>
namespace VECTOR
{
    class Vector
    {
    public:
        enum Mode {RECT, POL};
    // RECT for rectangular, POL for Polar modes
    private:
        double x;          // horizontal value
        double y;          // vertical value
        Mode mode;         // RECT or POL
    // private methods for setting values
        double get_mag(double a, double b);
        double get_ang(double a, double b);
        double get_x(double mag, double ang);
        double get_y(double mag, double ang);
    public:
        Vector();
        Vector(double n1, double n2, Mode form = RECT);
        void reset(double n1, double n2, Mode form = RECT);
        ~Vector();
        double xval() const {return x;}       // report x value
        double yval() const {return y;}       // report y value
        double magval() const {return get_mag(x, y);}   // report magnitude
        double angval() const {return get_ang(x, y);}   // report angle
        void polar_mode();                    // set mode to POL
        void rect_mode();                     // set mode to RECT
    // operator overloading
        Vector operator+(const Vector & b) const;
        Vector operator-(const Vector & b) const;
        Vector operator-() const;
        Vector operator*(double n) const;
    // friends
        friend Vector operator*(double n, const Vector & a);
        friend std::ostream & operator<<(std::ostream & os, const Vector & v);
    };

}   // end namespace VECTOR
#endif
```
---

在与另外一个驱动程序一起编译的时候编译器总是报错：
``` Cpp
error: passing 'const VECTOR::Vector' as 'this' argument of 'double VECTOR::Vector::get_mag(double, double)' discards qualifiers [-fpermissive]
```
弄得我真是头大，这error是哪里的问题？
可怜的我还是跑去爆栈上去找答案了，结果一搜就有，
传送：[<span class="fa fa-stack-overflow"></span> templates - C++ error: passing const as 'this' argument - Stack Overflow](http://stackoverflow.com/questions/23553859/c-error-passing-const-as-this-argument)
{% alert warning %}
原来是因为被声明成`const`的成员函数，只能在其中调用其他的`const`成员函数，不能是没有`const`关键字的成员函数！
{% endalert %}

我把函数都加上了`const`关键字，终于，C++的编译器终于不傲娇了！
