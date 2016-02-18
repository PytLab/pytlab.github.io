---
title: 动力学蒙特卡洛(kinetic Mont Carlo)基本原理小结
date: 2016-02-18 11:23:55
tags:
  - kMC
  - Monte Carlo
  - chemistry
  - catalysis
  - 学术
categories:
  - 学习小结
description: 在这里对动力学蒙特卡洛(kMC)的基本原理进行下简单总结。
mathjax: true
---

#### 克服模拟的时间尺度的局限
由于许多动态的过程(dynamic evolution)，例如表面的生长或者材料的老化，时间的跨度都在s以上，超出了分子动力学模拟的范围，动力学蒙特卡洛模拟便是针对这种长时间尺度的动态模拟的。
> Kinetic Monte Carlo attempts to overcome this limitation by exploiting the fact that the long-time dynamics of this kind of system typically consists of diffusive jumps from state to state. Rather than following the trajectory through every vibrational period, these state-to-state transitions are treated directly

![](PES.png)
<!-- more -->
kMC从某种程度上就是对MD的一种粗化，将关注点从**原子**粗化到**体系**，将**原子轨迹**粗化到**体系组态的跃迁**，那么模拟的时间跨度就将从原子振动的尺度提高到组态跃迁的尺度，这是因为这种处理方法摈弃了与体系穿越势垒无关的微小振动，而只着眼于体系的组态变化。因此，虽然不能描绘原子的运动轨迹，但是作为**体系演化**，其“组态轨迹”仍然是**正确**的。
此外，因为组态变化的时间间隔很长，体系完成的连续两次演化是独立的，无记忆的，所以这个过程是一种典型的**马尔可夫过程(Markov process)**，及从组态 $i$ 跃迁到组态 $j$ ，这一过程只与跃迁速率 $k\_{ij}$ 有关。

{% alert info %}
这里的跃迁速率可以理解为跃迁频率，或者说单位时间内发生组态跃迁的次数，能够衡量过程发生的难以程度。放在化学反应上就是反应速率常数<strong>k</strong>(跃迁次数/s)，有些文献上是用反应速率<strong>R</strong>表示的。
{% endalert %}

如果精确地知道 $k\_{ij}$，我们便可以构造一个随机过程，使得体系按照正确的轨迹演化。这里**正确**的意思是某条给定演化轨迹出现的几率与MD模拟结果完全一致(假设我们进行了大量的MD模拟，每次模拟中每个原子的初始动量随机给定)。这种通过构造随机过程研究体系演化的方法即为**动力学蒙特卡洛方法(kinetic Monte Carlo, KMC)**

---
#### kMC的时间步长
这里算是kMC中比较重要的一点。
$k\_{ij}$ 代表体系从组态 $i$ 逃逸到 $j$ 的速率，则发生跃迁的概率为 $k\_{tot} = \\sum\_{j}^{}{k\_{ij}}$
Gillespie给出的假设：
> Suppose that the system is in configuration $c$. The probability that a particular enabled reaction $c \\rightarrow c^{'}$ occurs in an infinitesimal period $\\delta t$ is given by $k\_{c \\rightarrow c^{'}}\*\\delta t$.

则体系处在某一状态且经过$\\delta t$之后体系状态未改变的概率为
$$P(T>= t + \\delta t) = P(T>= t)(1 - k\_{tot}\\delta t)$$
将等式写成：
$$\\frac{P(T>= t + \\delta t) - P(T>= t)}{\\delta t} = -k\_{tot}P(T>= t)$$
积分后得到**体系不发生跃迁的概率**
$$P(T>= t) = exp(-k\_{tot}t)$$
即，
$$p\_{survive} = exp(-k\_{tot}t)$$
则在时间t内体系发生跃迁的概率为
$$p\_{jump} = 1 - p\_{survive} = 1 - exp(-k\_{tot}t)$$
对时间$t$求导可得到体系发生跃迁的概率分布函数：
$$p(t) = k\_{tot}exp(-k\_{tot}t)$$

