---
title: KMCLib中的Interactions
date: 2016-03-29 10:08:48
tags:
  - KMCLib
  - Cpp
  - 学术
  - chemistry
  - catalysis
categories:
  - 学术
description: "Interactions类用于描述KMC过程中所有能够发生的process并对其进行操作，<br>其中包括process的抽取，更新process的match list等。"
feature: assets/images/features/kmclib.png
---

这两天由于一直在学习CMake相关的东西，没有把看的代码及时总结。特地现在补上。
Interaction类可以看作是process相应的集合，他在全局角度操作process。

### 成员数据
- `std::vector<Process> processes_`
    通过构造函数的参数传入，是所有能在lattice上面发生的process对象的集合。

- `std::vector<Process*> process_pointers_`
    存放指向processes_中process对象的指针，后面的函数都是在操作这个指针vector。

- `std::vector<std::pair<double,int> > probability_table_`
    这里叫做概率列表，其实里面并没有放于概率的数据，其中的pair的第一个entry放的是incremental rate，第二个entry放的是对应这个process能在当前lattice上面发生的位点的总数(也就是他的nSite数)。

- `bool implicit_wildcards_`
    是否使用通配符

- `bool use_custom_rates_`
    是否使用自定义速率

其他的几个成员数据目前还没有看到具体怎么用，先挖个坑，后面填。

<!-- more -->

### 成员函数
这里也只是拎出来几个重要的。
- `int Interactions::totalAvailableSites() const`
    计算整个lattice中能够发生process(无论是哪个process)的位点的总数。

- `void Interactions::updateProbabilityTable()`
    probability\_table\_这个变量在构造函数中初始化为与processes\_长度相同全部为<0.0, 0>的vector。
    这个函数就是使用两个迭代器遍历processes\_和probability\_table\_，然后将总速率叠加，并将相应的nSite填充到probability\_table\_中，结构我用下面的列向量来进行表示：
$$
\\left[\\begin{matrix}
     k\_{0} & n\_{0} \\\
     k\_{0} + k\_{1} & n\_{1}\\\
     k\_{0} + k\_{1} + k\_{2} & n\_{2}\\\
     \. & \.\\\
     \. & \.\\\
     \. & \.\\\
\\end{matrix} \\right]
$$

- `int Interactions::pickProcessIndex() const`
    此函数的作用是在probability_table中随机抽取一种process，并返回此process在列表中的索引号。
    这里算是kMC中比较重要的一个函数了，虽然代码不多，但是却可以在上面做文章。关键点就在于用什么算法进行查找并抽取事件。
    这里KMCLib的作者直接使用了STL中的lower_bound函数，此函数内部是通过二分查找实现定位，也就是说此查找的复杂度为 $O(logN)$

    ``` Cpp
    template <class ForwardIterator, class T>
    ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last, const T& val)
    {
        ForwardIterator it;
        iterator_traits<ForwardIterator>::difference_type count, step;
        count = distance(first,last);
        while (count>0)
        {
            it = first; step=count/2; advance (it,step);
            if (*it<val) {                 // or: if (comp(*it,val)), for version (2)
            first=++it;
            count-=step+1;
        }
        else count=step;
    }
    return first;
}
    ```
<br>
- `void Interactions::updateProcessMatchLists()`
    该函数的原型：
    ``` Cpp
    void Interactions::updateProcessMatchLists(const Configuration & configuration,
                                               const LatticeMap & lattice_map)
    ```
    此函数算是interaction类中最复杂的一个成员函数了。其作用是更新所有process的matchlist。通过将process的match list与configuration的minimal match list进行匹配，也就是使用两个迭代器分别遍历向量，判断entry是否相等，（根据MinimalMatchListEntry的运算符重载函数，相等也就意味着元素类型和相应的坐标都相等）。如果不相等就在process的matchlist中插入通配符`*`，若相等就要记录位置索引（因为插入通配符的原因，索引会发生偏移）。最后更新process中相关的信息即可。

    **我结合着其中的unittest来把这个东东的图像总结出来：**
    首先建个3x3x3的lattice map，初始化一个configuration，其中每个各点有2个basis_points，分别初始化为：
    ```
    (V, B), (V, B), (V, B), (V, B), ...
    ```
    其中的坐标分别为:
    ```
    ( (0.0, 0.0, 0.0), (0.3, 0.3, 0.3) ), (1.0, 1.0, 1.0), (1.3, 1.3, 1.3) ), ( (...), (...) ), ...
    ```
    可能的所有元素类型映射:
    ```
    "*": 0, "A": 1, "B":2, "C": 3
    ```

    在创建一个process,其中包含三个元素, 
    元素类型为：
    ```
    A, B, V --> B, A, A
    ```
    在执行此函数之后,程序会向process的match_list中插入相应的通配符,这样能够与configuration中minimal_match_list的顺序匹配上,当然直到process或者configuration的minimal_match_list的迭代器到达超尾位置,这种插入操作就终止了,然后更新process中的move_id相关信息.
    操作之后按照下图的顺序:
    ![](assets/images/blog_img/2016-03-29-KMCLib中的Interactions/lattice.png)
    则process的minimal_match_list元素顺序就更新为:
    ```
    A, V, *, *, *, B
    1, 3, 0, 0, 0, 2
    ```
