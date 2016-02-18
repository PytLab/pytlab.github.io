---
title: Kynetix新增动力学蒙特卡洛(kMC)计算模块
date: 2016-02-18 17:18:12
tags:
  - python
  - Kynetix
  - kinetics
  - kinetic model
  - catalysis
  - 学术
  - kMC
  - Monte Carlo
categories:
  - 学术
description: 给Kynetix新增了kMC模拟模块
---

目前[**Kynetix**](https://github.com/PytLab/Kynetix)已经可以进行动力学蒙特卡洛(kMC)模拟，其中kMC核心计算部分我将[**KMCLib**](https://github.com/leetmaa/KMCLib)整合到其中(若要使用Kynetix，请根据KMCLib的文档自行编译KMCLib)，这次更新的内容比较多，主要在以下几个地方进行了更新：
- `kMC Parser`, 负责将输入文件中的能量以及化学方程式进行解析
- `kMC Solver`, 这里是kMC模拟的核心，主要负责调用KMCLib的接口进行动力学蒙特卡洛计算，并对模拟进行On-the-fly分析（覆盖度，TOF等）
- `kMC Plotter`, 绘制表面形态的演化过程（可生成多图以及GIF动图）
<!-- more -->
---
下面简单介绍下kMC Solver。
`pynetics/solvers/kmc_solver.py`中包含两个类：
1. `KMCSolver`, 提供了根据提供的基元反应方程式和能量数据，使用过渡态理论以及碰撞理论等计算基元反应的反应速率等信息的接口。
2. `KMCLibSolver`, `KMCSolver`的子类，主要是使用KMCLib的接口生成相应KMCLib的对象和使用KMCLib进行计算。

---
`pynetics/solvers/kmc_plugins.py`中包含4个类：
1. `KynetixPlugin`, 是其他分析类的基类
2. `CoveragesAnalysis`, `KynetixPlugin`的子类，用于在蒙特卡洛循环中进行表面覆盖度的统计，可以进行**继续计算**.
3. `TOFAnalysis`, `KynetixPlugin`的子类，用于在蒙特卡洛循环中进行TOF统计计算, 可以进行**继续计算**。
4. `TOFCoveragesAnalysis`, `KynetixPlugin`的子类，同时进行覆盖度和TOF的统计，不支持**继续计算** 

---
`pynetics/solvers/kmc_functions.py`中包含分析的主要函数，独立出的原因是避免因为用户没有对该模块的C扩展进行编译而导致程序无法执行。

