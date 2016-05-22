---
title: Effective Python -- 列表排序中的优先级
date: 2016-05-22 16:33:05
tags:
  - Effective Python
  - python
categories:
  - 学习小结
feature: /assets/images/features/effectivepython.jpeg
description: "在《Effective Python》的第15条 “了解如何在闭包里使用外围作用域中的变量”中，作者使用了一个排序的例子，使用了list的sort方法中的key参数，由于我之前从来没有这样使用过，所以特地去查了下怎么使用排序关键字（sort key）。"
toc: true
---

### sort方法的参数

首先还是先看看python列表中`sort`方法的参数的作用。
``` python
In [9]: a = ['123', 'sd', 'asdfgf']

In [10]: a.sort??
Docstring: L.sort(key=None, reverse=False) -> None -- stable sort *IN PLACE*
Type:      builtin_function_or_method
```

sort在python3中有两个可选参数`key`, `reverse`。`reverse`参数就不多说了，那么`key`要怎么用？

官方文档中的解释为：
> key specifies a function of one argument that is used to extract a comparison key from each list element: `key=str.lower`. The default value is None (compare the elements directly).

也就是说`key`需要接受一个函数对象，一个可以作为排序依据的对象，这个函数将作用域列表中的所有元素，然后根据这个函数的返回值进行排序。
<!-- more -->
例如我将列表`a`根据字符串的长度排序，就要将函数`len`传给`key`，python就会根据这个函数的返回值将列表中的元素进行排序。
``` python
a
Out[13]: ['123', 'asdfgf', 'sd']

In [14]: a.sort(key=len)

In [15]: a
Out[15]: ['sd', '123', 'asdfgf']
```

这个函数的感觉就好象先对列表中的元素进行预处理，处理后再排序。

### 回到书中例子中

书中的例子大致描述下就是要对一个列表进行排序，例如：
``` pyhton
numbers = [8, 3, 1, 2, 5, 4, 7, 6]
```
但是如果其中的数字要是出现在另一个集合中就要进行优先排序：
``` python
group = {2, 3, 5, 7}  # 若列表中的元素是其中之一，就要先进行排列
```
因此要排序的话就要把出现在`group`中的元素优先级提高。这时就需要一个“预处理函数”将每个元素进行处理。
``` python
def sort_priority(values, group):
    def helper(x):
        if x in group:
            return (0, x)
        return (1, x)
    values.sort(key=helper)
```
这里作者使用了一个辅助函数来处理每个元素，这里他做的比较巧妙，就是判断元素是否在group中，然后在原有的元素前面添加一个数字0或者1，并返回tuple作为比较。
在这里举个例子，如果元素是3，它正好在group中，这样helper就会返回`(0, 3)`，然而如果是1，则会返回`(1, 1)`
这样把上面的列表中的元素全部预处理后，可以得到这样个列表：
``` python
[(1, 8), (0, 3), (1, 1), (0, 2), (0, 5), (1, 4), (0, 7), (1, 6)]
```
接下来python就需要对处理过以后的列表进行排序，这时就要涉及到python比较两个tuple的规则了。他首先比较元组中下标为0的元素，如果相等在比较下一个，以此类推。
这时候上面的列表中**第一个数字为0的肯定会排在前面，然后排在前面的几个数组在根据tuple中的第二个数进行排序**。这样就实现了有限排序。
``` python
In [19]: b = [(1, 8), (0, 3), (1, 1), (0, 2), (0, 5), (1, 4), (0, 7), (1, 6)]

In [20]: b.sort()

In [21]: b
Out[21]: [(0, 2), (0, 3), (0, 5), (0, 7), (1, 1), (1, 4), (1, 6), (1, 8)]
```
当然最终返回的还是排序过后的int列表而不是上面的tuple列表啦。但是顺序确实相同的。即，
``` python
In [23]: sort_priority(numbers, group)

In [24]: numbers
Out[24]: [2, 3, 5, 7, 1, 4, 6, 8]
```

