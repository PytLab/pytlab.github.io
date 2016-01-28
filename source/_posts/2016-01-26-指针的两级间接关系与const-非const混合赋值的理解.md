---
title: 指针的两级间接关系与`const`/非`const`混合赋值的理解
date: 2016-01-26 21:03:00
tags:
  - C
  - C++
  - 指针
categories:
  - 学习小结
description: " 对于指针的两级间接关系同`const`关键字赋值这部分，无论在看《C Primer Plus》还是《C++ Primer Plus》都是比较绕的地方，在这里我尝试简单梳理下关系。"
---
{% textcolor danger %}
为什么进入两级间接关系与一级间接关系不同, `const`与非`const`混合指针赋值方式将不再安全?
{% endtextcolor %}
<br>

#### 首先要明确一件事:
{% alert warning %}
非`const`指针是不能指向`const`值的。
{% endalert %}
原因很简单,非`const`指针可以改变指向的值,所以非`const`指针若指向`const`值,故可以改变`const`值,前后就矛盾了,因此这种赋值时禁止的。

<br>
#### 那再来看两级关系，

先声明两级关系的变量
``` Cpp
const int ** pp;
int * p;
const int n = 13;
```

<!-- more -->

先假设允许两级关系的赋值，也就是非`const`指针赋值给`const`指针的指针
``` Cpp
pp = &p; 
```

那么...
pp 做为`const`变量是可以与`const`值n关联的，即
``` Cpp
*pp = &n;
```
然后, 由于刚才`pp`指向了`p`， `p`指向的值就是`n`，则就意味着

**!!! `p`指向了`n` !!!**

这样一来，就和一开始说的情况一样了，我们将一个非`const`指针`p`指向了一个`const`值`n`，这样是禁止的。

所以，两级间接关系`const`与非`const`混合指针赋值方式将不安全。
