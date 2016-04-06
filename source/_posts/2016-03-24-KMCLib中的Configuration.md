---
title: KMCLib中的Configuration
date: 2016-03-24 20:23:52
tags:
  - Cpp
  - KMCLib
  - catalysis
  - chemistry
  - 学术
categories:
  - 学术
description: "KMCLib中的Configuration用于描述附着在晶格之上的元素相关的信息<br>包括他们的坐标，索引，ID号等"
feature: /assets/images/features/kmclib.png
toc: true
---

这个类由于在之前都有提及到，而且已经针对其中重要的成员函数进行了分析，在这里就不多扯了。
总之这个类是独立于晶格LatticeMap类的类型类，我还剩听佩服作者这样抽象的，这样如果后面我想加入多位点的话，也可以独立于晶格之外描述多位点的类，这样实现起来耦合量可以尽量减小。

其实这里我一直有个疑问，不知道看完全部代码的时候能不能找到答案，就是在`performProcess()`函数中：
``` Cpp
void Configuration::performProcess(Process & process,
                                   const int site_index,
                                   const LatticeMap & lattice_map)
```
根本就没有用到这个变量，为啥要传他进来？？
