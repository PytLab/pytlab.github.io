---
title: KMCLib中的match_list
date: 2016-03-24 09:05:54
tags:
  - Cpp
  - KMCLib
  - catalysis
  - chemistry
  - 学术
categories:
  - 学术
description: "match_lists以及minimal_match_list是KMCLib中链接configuration和process的桥梁，<br>同时还包含了元素类型、序号、坐标等信息，非常重要"
toc: true
---
match_list是由MinimalMatchList类对象组成的vector。
### MinimalMatchListEntry类
这其实是一个结构，用于存储每个位点相关的信息，或者说每个minimal_match_list中的元素。
- `bool has_move_coordinate;`
    用于标记这个元素是否移动，
    通常在创建process对象时候提供了move_origin和move_vector时会将此值设为true;

- `Coordinate move_coordinate;`
    这个元素如果有移动，他的移动向量，
    通常在创建process对象时候提供了move_origin和move_vector时会将此值设为move_vector中的值。

- `int match_type;`
    主要用于process中，存储process之**前**的元素类型int

- `int update_type;`
    主要用于process中，存储process之**后**的元素类型int

<!-- more -->

- `int index;`
    此处元素在全局中的索引值

- `int move_cell_i;`

- `int move_cell_j;`

- `int move_cell_k;`

- `int move_basis;`
    这个到底是有什么用处我现在还没看到，应该不是用于process中的

- `double distance;`
    此处元素相对中心元素的距离，通过坐标做差获取。

- `Coordinate coordinate;`
    这里的坐标是相对坐标，相对于中心坐标的
    （也就是minimal_match_list中的第一个MinimalMatchListEntry）

#### 关于MinmalMatchListEntry的一些操作符重载：
- `==`和`<`
    这两个主要是用于比较，其中优先比较距离中心元素的距离，若距离相同在进行坐标的比较。
    坐标是KMCLib中自定义的类，按照坐标的x, y, z依次进行比较。

- `!=`
    判别是否相等，若match_type为0，也就是这个entry中放的是通配符"*"则返回false，也就是不相等。
    否则按照match_type的值进行比较。
    如果类型也相同，就按照distance和坐标进行比较。
    比较的优先级顺序：
    1. 通配符
    2. 初始类型
    3. 距离
    4. 坐标

上面的运算符重载主要用于创建minimal_match_list之后进行排序，使得process的minimal_match_list和configuration的可以匹配上。


### Process类中的match_list
每个Process对象中都有一个相应的`minimal_match_list_`成员数据，他是一个`std::vector<MinimalMatchListEntry>`，里面放着与这个Process相关的每个元素的信息。

1. 由于Process是描述的前后元素的变化因此MinimalMatchListEntry对象中的`update_type`, `has_move_coordinate`, `move_coordinate`的值会特别设置用来描述元素变化的情况。

2. Process中的minimal_match_list是在构造函数中进行填充的。根据local_configuration的元素顺序进行填充。
    其中MinimalMatchListEntry中的has_move_coordinate成员和move_coordinate成员先初始化成默认数据false和Coordinate(0.0, 0.0, 0.0)。在后面再进行判断如果有传入move_origin和move_vector时再将相应的成员数据改成传入的数据。

3. 由于process只是局部信息，所以index成员全部设成-1，因此在process中的MinimalMatchListEntry中的index是个鸡肋。

4. 在最后将minimal_match_list中的数据填充完毕后，按照MinimalMatchListEntry升序进行排序，这也是为了和configuration中的match_list的顺序匹配上。

### Configuration类中的match\_list\_
Configuration这个类是用于描述全局类型信息的，其中的每个点都有一个自己的minimal_match_list合并起来形成一个vector就成为了Configuration类的match_lists。
因此类型为`std::vector< std::vector<MinimalMatchListEntry> > match_lists_;`

Configuration在构造函数中的成员初始化列表中通过resize来创建初始的列表，但是没有填充内容。

通过调用专门写的initMatchList()函数来初始化所有位置的minimal_match_list然后添加到match_lists中。这个函数并不是什么重要的函数，只是一个遍历操作而已，关键的任务在minimalMatchList()身上。

#### minimalMatchList()函数
这个函数有重载版本，另一个是用于返回值的const成员函数，用于返回相应位置上的minimal_match_list函数。
关键的函数在于另一个版本的这个：
``` Cpp
const std::vector<MinimalMatchListEntry> & minimalMatchList(const int origin_index,
                                                            const std::vector<int> & indices,
                                                            const LatticeMap & lattice_map) const;
```
此函数返回相应origin_index元素为中心的minimal_match_list。参数indices是他的neighbor indices，初始顺序为正常顺序。

从indices中使用迭代器进行遍历，填充数据：
- match_type: 当前位置的元素类型
- update_type: 此数据在configuration中没有作用，设为-1
- distance: 相对于中心坐标的距离。
- coordinate: 相对于中心坐标的坐标（**因此凡是在minimal_match_list中的坐标和距离全部都是相对于中心点的相对值**）
- index: 这个值在这里就有用了，用来标记此元素在全局中的索引。

当然最后还是要进行升序排序，为了与process中的match_list位置能够对应上。
