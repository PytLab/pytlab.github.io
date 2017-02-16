---
title: ECMAScript中的作用域链和闭包
date: 2017-02-14 22:47:33
tags:
 - JavaScript
 - ECMAScript
categories:
 - 学习小结
feature: /assets/images/blog_img/2017-02-14-ECMAScript中的作用域链和闭包/scope_chain_1.png
toc: true
---

本文对ECMAScript中的作用域以及闭包的概念进行一下小结。主要是整理概念便于理解和回忆。

## 作用域
任何程序设计语言都有作用域的概念，简单的说，作用域就是变量与函数的可访问范围，即作用域控制着变量与函数的可见性和生命周期。

## 函数创建和调用时的作用域链
函数和其他对象一样，都拥有
1. 可以直接通过代码访问的属性
2. 一些仅供js引擎访问的内部属性(用`[[...]]`扩起来的那些)。

<!-- more -->

### 函数创建
``` javascript
function add(num1, num2){
    var sum = num1 + num2;
    return sum;
}
```

`add`函数在创建的时候在函数对象内部会先创建一个预先包含全局变量对象的作用域链，这个作用于链内部属性`[[Scope]]`中。

![](/assets/images/blog_img/2017-02-14-ECMAScript中的作用域链和闭包/scope_chain_0.png)
<p style="text-align: center">*图片来自网络*</p>

### 函数执行
当`add`函数调用时，例如如下的调用：
``` javascript
var total = add(5, 10);
```

#### 1. 创建执行环境
会为函数创建一个内部对象-**执行环境**,每个执行环境都有自己的**作用域链**用于标识符解析。
#### 2. 为执行环复制原型链
复制函数对象的`[[Scope]]`属性中的对象(原型链)构建**属于执行环境**的作用域链, 这时候执行环境的原型链相当于函数对象原型链的一个**副本**。
#### 3. 创建活动对象(activation object)
复制完原型链后，创建一个叫活动对象的对象，该对象包含了当前函数中的所有局部变量、名称参数、参数集合以及this。
#### 4. 将活动对象放到作用域链的最前端
将活动对象当作变量对象放到作用域链的最前端。

![](/assets/images/blog_img/2017-02-14-ECMAScript中的作用域链和闭包/scope_chain_1.png)
<p style="text-align: center">*图片来自网络*</p>

## 相关概念总结
个人浅显总结如有理解错误欢迎指正。
### 执行环境
**执行环境(execution context)**定义了变量或者函数有权访问的其他数据，是在函数调用时候创建的一个对象，包含有自己的作用域链。
每个函数都有自己的执行环境，当执行进入一个函数的时候，函数环境就被压入环境栈中，函数体退出后环境会在栈中弹出，把控制权返回给之前的执行环境。ECMAScript程序中的执行流正是通过这个机制控制者的。

### 环境变量
每个执行环境都有一个**变量对象(variable object)**与之相关联，即执行环境中的变量和函数都保存在这个对象中。
我们无法直接获取变量对象这个对象本身并进行操作，但是js解释器会在我们处理变量函数等数据的时候在后台调用它。

### 作用域链
简单的讲，作用域链就是将多个变量对象进行链接起来的东东，这样能够保证执行环境能够有序的访问所能访问到的变量和函数。
这个链的**最前端**始终都是**当前执行代码所在的变量对象**，如果当前环境是函数的话，那么当前的**活动对象(activation object)**就做为变量对象作为作用域链的最前端。
{% alert info %}
作用域链的本质是一个指向变量对象指针的列表，只是引用不包含实际变量对象。
{% endalert %}

### 标识符解析
标识符解析是沿着作用域链一级一级搜索的。搜索过程从链的最前端逐级向后知道找到标识符或者找不到标识符抛出错误。
凡是能访问到的变量就说在作用域链中找到了这个变量。

## `with`语句改变作用域链
``` javascript
function initUI(){
    with(document){
        var bd=body,
        links=getElementsByTagName("a"),
        i=0,
        len=links.length;
        while(i < len){
            update(links[i++]);
        }
        getElementById("btnInit").onclick=function(){
            doSomething();
        };
    }
}
```
向上面的代码，通过当代码运行到`with(document)`的时候，`initUI`的运行环境中的作用域链被改变了，改变如下：
1. 创建一个新的变量对象，并将`document`变量对象中的所有属性添加进去
2. 将这个新的变量对象放到执行环境作用域链的头部。

![](/assets/images/blog_img/2017-02-14-ECMAScript中的作用域链和闭包/scope_chain_2.png)
<p style="text-align: center">*图片来自网络*</p>

## 闭包
了解了上面的执行环境，变量对象、活动对象，作用域链的概念以后闭包就很容易理解了。

就是如果在一个函数内部定义的函数的作用域链中包含创建他的外层函数的活动对象，即使外层函数返回，外层函数的执行环境销毁，但是他的活动对象仍然被内部的函数的作用域链引用着，因此不会被销毁。

## 参考
- [JavaScript 开发进阶：理解 JavaScript 作用域和作用域链](http://www.cnblogs.com/lhb25/archive/2011/09/06/javascript-scope-chain.html)
- 《JavaScript高级程序设计(第3版)》

