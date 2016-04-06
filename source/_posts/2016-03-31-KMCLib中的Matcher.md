---
title: KMCLib中的Matcher
date: 2016-03-31 16:17:06
tags:
  - KMCLib
  - Cpp
  - Monte Carlo
  - chemistry
  - catalysis
  - 学术
categories:
  - 学术
feature: /assets/images/features/kmclib.png
description: "KMCLib中的Matcher类主要用于将所有process与configuration进行匹配，<br>并更新process中相关的信息，例如能发生反应的sites、总速率等"
---

这个类算是KMCLib中最后一个大类了，也回答了我在看代码之初有的很多疑问，例如如何更新匹配相关的信息，如何更新事件列表等。
这个类是没有成员数据的，可以把`Matcher`类看作是与匹配相关的函数的封装。

但是在头文件中却定义了两个相关的结构来存储操作信息：
``` Cpp
/// A minimal struct for representing a task with a rate.
struct RateTask
{
    int index;
    int process;
    double rate;
};


/// A minimal struct for representing a remove task.
struct RemoveTask
{
    int index;
    int process;
};
```
<!-- more -->

### 成员函数
- `void calculateMatching()`
    原型：
    ``` Cpp
    void calculateMatching(Interactions & interactions,
                           Configuration & configuration,
                           const LatticeMap & lattice_map,
                           const std::vector<int> & indices) const;
    ```
    此函数就把参数中的indices对应的元素用所有的process进行匹配，从而将process中的信息进行更新，例如有些点已经不能发生此process则要针对这个process中进行删除操作。执行这个函数后interactions中的process都会进行更新。

- `void Matcher::matchIndicesWithProcesses()`
    原型：
    ``` Cpp
    void Matcher::matchIndicesWithProcesses(const std::vector<std::pair<int,int> > & index_process_to_match,
                                            const Interactions  & interactions,
                                            const Configuration & configuration,
                                            std::vector<RemoveTask> & remove_tasks,
                                            std::vector<RateTask>   & update_tasks,
                                            std::vector<RateTask>   & add_tasks) const
    ```
    此函数主要用于从`index_process_to_match`中根据匹配的情况将其分类到不同的任务中，例如本身不能匹配的但是现在能匹配上的就执行添加操作，就将位点和process的index以及process的速率放入到add_tasks中。其他情况也是类似的。这样执行完这个函数，remove_tasks, update_tasks, add_tasks将会被填充。

- `void Matcher::updateProcesses()`
    原型：
    ``` Cpp
    void Matcher::updateProcesses(const std::vector<RemoveTask> & remove_tasks,
                                  const std::vector<RateTask>   & update_tasks,
                                  const std::vector<RateTask>   & add_tasks,
                                  Interactions & interactions) const
    ```
    此函数读取上面的三个tasks列表，根据列表进行相应的process更新操作，主要是更新process的site和rate等成员数据的值。函数执行后，相应的process将全部更新。

此函数还有两个函数分别是
- `void Matcher::updateRates()`

- `double Matcher::updateSingleRate()`

这是个回调函数，具体用途上没发现，后面进行补充。
