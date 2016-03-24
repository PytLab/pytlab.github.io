---
title: KMCLib中的LatticeMap
date: 2016-03-21 21:47:42
tags:
  - Cpp
  - KMCLib
  - catalysis
  - chemistry
  - 学术
categories: 
  - 学术
description: "KMCLib中的LatticeMap类<br>主要用于描述整个3维网格，其中包括网格格点的各种操作，如移动，寻找NN等等"
feature: assets/images/features/kmclib.png
---
这里的成员数据全部`protected`
### 成员数据
``` Cpp
int n_basis_;                   // 基点个数

std::vector<int> repetitions_;  // site各个方向上重复次数

std::vector<bool> periodic_;    // 周期性边界条件
```
这个类的成员数据相对比较少，就三个，就可以描述整个三维网格的所有性质了。
其中后两个好理解，但是第一个我看了许久代码都不知道是个啥东西，还好曹老师给的指点，终于弄明白了所谓基点数目和基点到底是什么东西。
<!-- more -->
#### 关于`basis_points`和`n_basis`
其实也没啥好说的，本身没有什么难理解的，只要知道是怎么操作的了就一切都好说了。先上图吧：
![](assets/images/blog_img/2016-03-21-KMCLib中的LatticeMap/lattice_indices.png)
上图描述了KMCLib中三维网格模型的基本信息（忽略我拙劣的画风 -\_-！），其中包括：
- 三个基向量的方向和顺序
- 每个位点的索引号和顺序
- 基点的信息


从上图中可以看出，所谓基点数目就是小括号中的数目，虽然目前还没看到这样做的作用是什么，不过好像的确是能够通过这种方式模拟多位点如hollow、top、bridge这种，现在概念都明了了，代码看起来速度可以飞起了，嗖嗖嗖～～===3333

### 成员函数
这里也只是罗列几个比较重要的方法
类方法实现代码中有一个static变量，用于存储含有不同n_basis的index
``` Cpp
// Temporary storage for the indices form cell.
static std::vector<int> tmp_cell_indices__;
```

- `indicesFromCell()` 和 `indexToCell()`
    ``` Cpp
    const std::vector<int> & LatticeMap::indicesFromCell(const int i,
                                                         const int j,
                                                         const int k) const
    { 
        ... 
    }

    void LatticeMap::indexToCell(const int index,
                                 int & cell_i,
                                 int & cell_j,
                                 int & cell_k) const
    { 
        ... 
    }
    ```

    这两个函数是可逆的，第一个是提供位点在x、y、z三个方向上分别的序号，返回全局索引序号；第二个则是相反的操作，提供全局索引，返回相对应的在三个方向上的分量索引值。具体数据可以去看KMCLib的unittest代码 [<span class="fa fa-github"></span> KMCLib/test_latticemap.cpp at master-pytlab · PytLab/KMCLib](https://github.com/PytLab/KMCLib/blob/master-pytlab/c%2B%2B/unittest/test_latticemap.cpp)

- `indexFromMoveInfo()`
    ``` Cpp
    int LatticeMap::indexFromMoveInfo(const int index,
                                      const int i,
                                      const int j,
                                      const int k,
                                      const int basis) const
    { 
        ... 
    }
    ```

    此函数是用于返回某个位点加上某个向量以后新位点的索引，虽然还没看到此函数用在什么地方，不出意外是用于元素在网格中移动的时候用的。
    具体的操作：
    例如最开始一个site的索引值为16，这个网格的基点数目是3，也就是说每个点上的索引都是一个三维向量例如`(15, 16, 17)`，则参数中index就为`16`;
    接下来的三个参数代表位移向量在三个方向上的分量`(i, j, k)`;
    最后是`basis`这个参数有点不好理解，它指的是在basis_point内的相对于初始位置的偏移量，例如最开始是`(15, 16, 17)`中的`16`，经过向量的位移之后，到达全局索引`(18, 19, 20)`但是还是指向的中间的19，因此若需要一个相对的偏移量指向第一个的话，则就需要`basis`这个值了，他的值应该为-1。

- `neighbourIndices()`和`supersetNeighbourIndices()`

    ``` Cpp
    std::vector<int> LatticeMap::neighbourIndices(const int index,
                                                  const int shells) const
    { 
        ... 
    }

    std::vector<int> LatticeMap::supersetNeighbourIndices(const std::vector<int> & indices,
                                                          const int shells) const
    { 
        ... 
    }
    ```
    这两个函数相似，后者就是循环调用前者然后去重的产物。至于所谓的neighbor，还是先上图吧：
    ![](assets/images/blog_img/2016-03-21-KMCLib中的LatticeMap/neighbors.png)
    所谓找neighbor就是将shell包含的内部全部site的索引数按顺序返回出来。如果有周期性边界条件的限制，例如a和b轴都没有周期性边界条件，寻找0位置的neighbor，则按照图上的顺序返回neighbor的全局索引。
