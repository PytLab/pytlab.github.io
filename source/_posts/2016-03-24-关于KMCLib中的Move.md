---
title: 关于KMCLib中的Move
date: 2016-03-24 14:34:19
tags:
  - Cpp
  - KMCLib
  - catalysis
  - chemistry
  - 学术
categories:
  - 学术
description: "关于KMCLib中的MOVE相关的东西涉及到坐标和各种索引，这里比较混乱我觉得还是有必要梳理下。"
toc: true
---
move来源于process并作用于configuration。

### Process中的move相关
#### Process中与move相关的变量：
- `std::vector<int> move_origins`
    说明在process中那些元素是在process发生时有移动的，(0, 1)的话就是指第一个（中心）和第二个元素发生了移动。

- `std::vector<Coordinate> move_vectors`
    对应move_origin的元素移动的向量。

- `std::vector< std::pair<int,int> > id_moves_`
    用于描述元素移动前后位置的变化，通过元素在process中的minimal_match_list中的索引号来表示，例如
    id\_moves\_[0]中存放着两个值 (1, 4)，则就说明在minimal_match_list中的第1号元素（也就是第二个）通过process的作用之后会移动到第4号位置上。
<!-- more -->
    变化如下图所示：
    ![](assets/images/blog_img/2016-03-24-关于KMCLib中的Move/move.png)
    因此对应上图的id\_moves\_的内容为：
    ``` shell
    <1, 4>, <4, 2>, <2, 0>, <0, 1>
    ```

#### Process中相关的成员方法
有process只是描述变化的信息，因此只要把移动相关的索引值和移动向量添加到process的minimal_match_list中即可，麻烦的操作在configuration那里。

### Configuration中的move相关
#### Configuration与move相关的成员变量
- `int n_moved_`
    记录在一次process执行时有多少元素发生了改变（只要是前后元素类型不一致且不是通配符的时候，这个值就自加1，每次执行process时这个值会先重置。

- `std::vector<int> moved_atom_ids_`
    记录在process中有移动过的元素的全局id号，例如若其中的值为：
    ``` Cpp
    | 1434 | 150 | ...
    ```
    则代表id为1434和150的元素发生了移动

- `std::vector<Coordinate> recent_move_vectors_`
    存储对应moved\_atom\_ids元素的移动向量值

- `std::vector<std::string> elements_`
    与move间接相关，以为位置的移动会改变elements的排列。

- `std::vector<int> types_`
    与上一条的原因相同

- `std::vector<std::string> atom_id_elements_`
    这个值其实与移动无关，毕竟移动并不会改变id对应的元素类型，但是process仍然会造成这个值的改变，不是移动，而是通过直接的替换。

- `std::vector<int> atom_id_`
    因为移动，所以必定会造成id之间的互换。

- `std::vector< std::vector<MinimalMatchListEntry> > match_lists_`
    全局晶格的元素发生了移动或者替换，必定需要更新configuration的match\_lists\_中的数据，但是其实只需要更新match_type就好了，位置不会变（晶格没有变动），configuration的minimal\_match\_list不存储移动相关的数据。

#### Configuration与move相关的成员函数
- performProcess()
    ``` Cpp
    void Configuration::performProcess(Process & process,
                                       const int site_index,
                                       const LatticeMap & lattice_map)
    ```

    不得不吐槽这个函数让我看了很久，不过也要感谢这个函数，他让我把KMCLib的最难理解的部分搞清楚了。
    这个函数中同时移动着**5个**迭代器
    ``` Cpp
    // 遍历process中的minimal_match_list
    auto it1 = process_match_list.begin();

    // 遍历全局晶格中某点的minimal_match_list
    auto it2 = site_match_list.begin();

    // 下面三个都用于用记录信息的
    auto it3 = process.affectedIndices().begin();
    auto it4 = moved_atom_ids_.begin();
    auto it5 = recent_move_vectors_.begin();
    ```

<br>
#### 我在这里决定把这个函数贴上来通过注释的方式解读这段代码：
``` Cpp
void Configuration::performProcess(Process & process,
                                   const int site_index,
                                   const LatticeMap & lattice_map)
{
    // PERFORMME
    // Need to time and optimize the new parts of the routine.

    // Get the proper match lists.
    // 同时获取相对应的两个minimal_match_list的迭代器用于遍历容器
    const std::vector<MinimalMatchListEntry> & process_match_list = process.minimalMatchList();
    const std::vector<MinimalMatchListEntry> & site_match_list    = minimalMatchList(site_index);

    // Iterators to the match list entries.
    std::vector<MinimalMatchListEntry>::const_iterator it1 = process_match_list.begin();
    std::vector<MinimalMatchListEntry>::const_iterator it2 = site_match_list.begin();

    // Iterators to the info storages.
    std::vector<int>::iterator it3 = process.affectedIndices().begin();
    std::vector<int>::iterator it4 = moved_atom_ids_.begin();
    std::vector<Coordinate>::iterator it5 = recent_move_vectors_.begin();

    // Reset the moved counter.
    n_moved_ = 0;

    // Loop over the match lists and get the types and indices out.
    for( ; it1 != process_match_list.end(); ++it1, ++it2)
    {
        // Get the type out of the process match list.
        // 这里应该被更新的元素类型，
        // 如果process没有改变类型则与(*it2).match_type的值是相同的
        const int update_type = (*it1).update_type;

        // Get the index out of the configuration match list.
        // 这个元素在全局晶格中的索引值，因为之后的所有操作都离不开这个索引值来定位，
        // 都是在这个位置上进行一系列操作的
        const int index = (*it2).index;

        // NOTE: The > 0 is needed for handling the wildcard match.
        // 如果类型的确是发生了改变，就需要更新一系列的数据
        if (types_[index] != update_type && update_type > 0)
        {
            // Get the atom id to apply the move vector to.
            // 获取原本位置上的元素的id号
            const int atom_id = atom_id_[index];

            // Apply the move vector to the atom coordinate.
            // 由于发生了移动，这个id的元素的位置要进行更新
            atom_id_coordinates_[atom_id] += (*it1).move_coordinate;

            // Set the type at this index.
            // 修改全局晶格的信息
            // 以前晶格位置上的类型进行更新
            types_[index]    = update_type;  
            elements_[index] = type_names_[update_type];

            // Update the atom id element.
            // 这里主要是针对直接替换掉的元素而不是移动造成的元素变更
            // 正常通过移动是无法改变id对应元素类型的
            if (!(*it1).has_move_coordinate)
            {
                atom_id_elements_[atom_id] = elements_[index];
            }

            // Mark this index as affected.
            // 下面这些都是记录一些信息

            // 这个process改动了的全局晶格索引
            (*it3) = index;
            ++it3;

            // Mark this atom_id as moved.
            // 哪些元素被移动了
            (*it4) = atom_id;
            ++it4;
            ++n_moved_;

            // Save this move vector.
            // 保存相应元素的移动坐标，如果是直接替换这里应该是(0.0, 0.0, 0.0)
            (*it5) = (*it1).move_coordinate;
            ++it5;
        }
    }

    // 下面的操作时用来更新atom_ids_
    // 由于元素通过process进行了位置的移动，相应的atom_ids中的顺序也要进行及时的更新
    // Perform the moves on all involved atom-ids.
    const std::vector< std::pair<int,int> > & process_id_moves = process.idMoves();

    // Local vector to store the atom id updates in.
    // 这个pair里面放了关于什么元素移动到了哪里的信息，例如
    // <元素id, 移动到的晶格索引号（也就是位置）>
    std::vector<std::pair<int,int> > id_updates(process_id_moves.size());

    // Setup the id updates list.
    for (size_t i = 0; i < process_id_moves.size(); ++i)
    {
        const int match_list_index_from = process_id_moves[i].first;
        const int match_list_index_to   = process_id_moves[i].second;

        // 获取在全局晶格中跳动的起始位置和终点位置（这里都是用索引号确定的）
        const int lattice_index_from = site_match_list[match_list_index_from].index;
        const int lattice_index_to   = site_match_list[match_list_index_to].index;

        id_updates[i].first  = atom_id_[lattice_index_from];  // 什么元素（id号）
        id_updates[i].second = lattice_index_to;              // 移动到了全局的哪里（全局索引号）
    }

    // Apply the id updates on the configuration.
    for (size_t i = 0; i < id_updates.size(); ++i)
    {
        const int id    = id_updates[i].first;
        const int index = id_updates[i].second;

        // Set the atom id at this lattice site index.
        atom_id_[index] = id;                      // 跳到的地方把元素id更新
        atom_id_elements_[id] = elements_[index];  // 更新元素相应的元素类型，感觉没必要？

    }
}
```

- `void Configuration::updateMatchList(const int index)`
    process执行过后需要对configuration的match\_lists\_进行更新，但是这里只需要对里面的match_type进行更新就可以了。
    我觉得这里有空间降低复杂度，后面开始写新代码的时候会考虑修改这里。
