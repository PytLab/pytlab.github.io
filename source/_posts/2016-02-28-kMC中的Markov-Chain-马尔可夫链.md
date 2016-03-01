---
title: kMC中的Markov Chain(马尔可夫链)
date: 2016-02-28 21:10:01
tags:
  - kMC
  - Monte Carlo
  - catalysis
  - 学术
  - LinearAlgebra
  - Kynetix
categories:
  - 学习小结
description: "在这里总结下kMC中的数学原理，均为本人的理解和原创，如有错误欢迎指正。"
toc: true
---
在动力学蒙特卡洛kMC模拟中，因为组态变化的时间间隔很长，体系完成的连续两次演化是独立的，无记忆的，所以这个过程是一种典型的马尔可夫过程(Markov process)。
下面先定义下符号：
{% alert info %}
<br>
configuration: $u, v, ... $
<br>
event: $ \alpha, \beta , ... $
<br>
transition rate($v \rightarrow u$) : $ \omega_{u v} $
{% endalert %}

### Master Equation
体系的时间演变通过**Markov master equation**来描述
$$ \\bar{p}\_{u}(t) = \\sum\_{v}^{}{\\omega\_{u v}p\_{v}(t) - \\omega\_{v u}p\_{u}(t)} $$
其中${p}\_{u}(t)$是体系在时间$t$处在$u$的概率。
<!-- more -->

### Transition Matrix
类似统计力学中的Probability Vector， kMC中也应该有一个向量来表示体系的概率状态，在这里我用一个向量$\\pi(t)$表示
$$
\\pi(t) = \\left[\\begin{matrix}
         p\_{u}(t)\\\
         p\_{v}(t)\\\
         p\_{o}(t)\\\
         \.\\\
         \.\\\
         \.\\\
    \\end{matrix} \\right]
$$
其中向量中的entry中${p}\_{u}(t)$表示体系在时间$t$处在$u$的概率
则整个kMC的体系的演化过程也就是这个状态向量的演变过程。有向量的演变，则必须有相应的转移矩阵的存在即所谓的markov矩阵，Transfer/Transition Matrix。
在这里transition matrix用$\\omega$表示
$$
\\omega = 
    \\left[\\begin{matrix}
         k\_{u \\rightarrow u} & k\_{v \\rightarrow u} & k\_{o \\rightarrow u} & ...\\\
         k\_{u \\rightarrow v} & k\_{v \\rightarrow v} & k\_{o \\rightarrow v} & ...\\\
         k\_{u \\rightarrow o} & k\_{v \\rightarrow o} & k\_{o \\rightarrow o} & ...\\\
         \. & \. & \. & ...\\\
         \. & \. & \. & ...\\\
         \. & \. & \. & ...\\\
    \\end{matrix} \\right]
$$
若模拟的网格的大小为$300 \times 300$，且每个位点可能的状态为$3$个（例如CO氧化反应），则方阵$\\omega$的为$3^{n \times n}$阶方阵，总共有$3^{2n^{2}}$个元素，可见如果要模拟的体系很大的时候，这个矩阵式非常非常非常的大的（重要的事情重复三遍）！

矩阵中的每一列针对的是起始于同一种configuration，每一行针对的是同一种终态configuration。
例如transfer matrix中的第$u$列的列向量$V\_{u}$就等于configuration为$u$时的事件的列表，
$$
V\_{u} = \\left[\\begin{matrix}
         \\alpha\_{u \\rightarrow u}\\\
         \\alpha\_{u \\rightarrow v}\\\
         \\alpha\_{u \\rightarrow o}\\\
         \.\\\
         \.\\\
         \.\\\
    \\end{matrix} \\right]
    =
    \\left[\\begin{matrix}
         k\_{u \\rightarrow u}\\\
         k\_{u \\rightarrow v}\\\
         k\_{u \\rightarrow o}\\\
         \.\\\
         \.\\\
         \.\\\
    \\end{matrix} \\right]
$$
其中不为0的entry为可发生事件的发生速率。

### Ensemble Average
通过多个trajectory可以得到多个体系的演化过程，大量的系综平均可以得到近似的$p\_{u}(t)$值.
$$ 
traj(i): \\pi\_{1}^{i} \\rightarrow \\pi\_{2}^{i} \\rightarrow \\pi\_{2}^{i} \\rightarrow ... \\\  
traj(j): \\pi\_{1}^{j} \\rightarrow \\pi\_{2}^{j} \\rightarrow \\pi\_{2}^{j} \\rightarrow ... \\\
traj(s): \\pi\_{1}^{s} \\rightarrow \\pi\_{2}^{s} \\rightarrow \\pi\_{2}^{s} \\rightarrow ... \\\
\.\\\
\.\\\
\.
$$  

### 总结
因此，通过状态向量和转移矩阵就可以构成kMC中的Markov链，当然这只是将kMC过程抽象出来的理论模型，由于转移矩阵过于庞大，只能用所谓的trajectory的方法来做系综平均的方法来进行kMC模拟。

