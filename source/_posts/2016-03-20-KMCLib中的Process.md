---
title: KMCLib中的Process
date: 2016-03-20 21:22:41
tags:
  - Cpp
  - KMCLib
  - catalysis
  - chemistry
  - 学术
catogories:
  - 学术
description: "KMCLib C++部分的Process类, <br>此类主要用于描述KMC过程中局部反应发生的过程，同时也包含了该过程在全局网格中的统计结果，也就是匹配结果。"
toc: true
---
### 成员数据
此类的成员数据均为`protected`
- `int process_number_;`  
    该过程ID号；

- `int range_;`          
    还未完全弄清楚其作用，是和坐标有关的一个范围，后续添加。

- `double rate_;`         
    这没什么好说的，反应速率；

- `double cutoff`
    和坐标距离相关，数值上应该等于process涉及到的点到中心
    也就是process中第一个点）的最大距离，
    尚未弄清求作用；

- `std::vector<int> sites_;`
    在整个网格中能和此反应匹配上的位点的向量；

- `std::vector<MinimalMatchListEntry> minimal_match_list_; `
    用于存储每个位点信息的数据结构；

- `std::vector<int> affected_indices_;`
    查看process前后每个位点类型是否相同
    如果不同则将0追加到其末尾
    (2016-03-31更新)
    这个变量主要用于更新process发生后确定发生process的index周围有哪些位置**受影响**,
    所谓受影响，也就是指由于该点由于参与了反应，元素的类型会发生变化，他的match_list也就相应的发生变化，也导致原本能够匹配的process不再能够在该点匹配或者原本不能匹配的process可以在该点匹配了。这个信息主要是用于Matcher类中。

<!-- more -->

- `std::vector<int> basis_sites_`
    这个用于描述这个process可以用在basis_sites中的哪一个点上。
    例如若此变量的值为(0, 1)，则这个process可以用在每个latticesite上的第一个和第二个点，若有第三个点则不能作用在第三个点上

- `std::vector< std::pair<int,int> > id_moves_;`
    记录这个process中涉及到元素移动的信息。
    其中每个pair的第一个值放置元素的初始位置（process中的位置），第二个元素放置移动到的目标位置。
    例如，有个元素从1号位置移动到了4号位置，则pair中的数据就为(1, 4)

<br>
### 成员方法
这里只罗列几个重要的
``` Cpp
/*
大致过程是
- 先创建两个局部的Configuration对象，
- 针对每个Configuration对象进行遍历，
  将元素信息都生成一个`MinimalMatchListEntry`对象然后在添加到`minimal_match_list_`变量中，
- 最后再处理和移动向量相关的操作，具体的用途还不清楚，感觉可能与扩散的模拟相关；
*/
Process(const Configuration & first,
        const Configuration & second,
        const double rate,
        const std::vector<int> & basis_sites,
        const std::vector<int> & move_origins=std::vector<int>(0),
        const std::vector<Coordinate> & move_vectors=std::vector<Coordinate>(0),
        const int process_number=-1);

// 返回在整个网格中该过程的所有速率总和；
virtual double totalRate() const

// 将编号为index的site加入到process的`sites_`中，说明在该点上能够与该过程匹配成功。
virtual void addSite(const int index, const double rate=0.0)

// 前面的逆过程；
virtual void removeSite(const int index)

// 在`site_`中随机抽取一个site；
virtual int pickSite() const
```

<br>
#### 亮点
在`virtual void removeSite(const int index)`中作者有个小技巧降低了删除的时间复杂度：
``` Cpp
void Process::removeSite(const int index)
{
    // Find the index to remove.
    std::vector<int>::iterator it1 = std::find(sites_.begin(),
                                               sites_.end(),
                                               index);
    // Swap the index to remove with the last index.
    std::vector<int>::iterator last = sites_.end()-1;
    std::swap((*it1), (*last));
    // Remove the last index from the list.
    sites_.pop_back();
}
```
STL的`vector`内部是使用数组来存储数据的而不是链表，因此插入和删除的平均时间复杂度是$O(N)$，因此`erase()`方法的时间复杂度为$O(N)$，但是作者的做法通过先将要删除的元素与最后一个元素对调然后再用`pop_back()`方法删除末尾元素，通过两个复杂度为常数$O(1)$的操作将整体复杂度也降到了常数级。

