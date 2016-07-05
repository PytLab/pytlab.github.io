---
title: 发现KMCLib中一个隐藏bug
date: 2016-07-05 19:45:38
tags:
 - KMCLib
 - catalysis
 - kMC
 - Monte Carlo
categories:
 - 学术
description: "发现了一个很难发现的KMCLib中的bug，只需要修改以个参数就可以修复，但是很难发现。"
feature: /assets/images/features/kmclib.png
---
由于为了看自己的KMCLibX是不是能跑出正确的结果，我将每一个蒙特卡洛步的configuration进行了散点图，来观察每一步事件的选取和configuration发生的变化，把事件全部定义好以后，跑了500步做了下图就发现，吸附在top位的O有落单的现象，明明我没有在事件中定义过有破坏平躺吸附在表面的氧原子的事件啊，这就奇怪了。于是我就将做图的范围缩窄，逐渐定位到了是哪一步开始出现单个top位上的氧原子。
如下图，是发生在第27步的时候，直接会有一个氧分子直接变成吸附在bridge site上的CO：
![](/assets/images/blog_img/2016-07-05-发现KMCLib中一个隐藏bug/configuration26.png)
![](/assets/images/blog_img/2016-07-05-发现KMCLib中一个隐藏bug/configuration27.png)

这看着就知道是一个CO吸附，但是强制发生了，也就是说明在执行第27步的时候，第225个点是在process28中的（别问我为什么知道是第几个事件第几个位点上，亲自写代码自然会知道）。所以这就是一个bug，在进行事件发生后的局部收到影响位点上没能够正确的重新进行匹配(re-matching)。

找来找去，原来发现原始的KMCLib中在singleStep()中执行每一步选中的事件之后会保存所谓的AffectedIndices说明这些点在上次迭代的时候发生了改变，那么这些点的某一范围内的点都应该对所有事件重新进行匹配来更新事件中的相应数据。那么关键就在于这个**范围**了，原始代码中是这样的：
``` Cpp
void LatticeModel::singleStep()
{
    // Select a process.
    Process & process = (*interactions_.pickProcess());

    // Select a site.
    const int site_index = process.pickSite();

    // Perform the operation.
    configuration_.performProcess(process, site_index);

    // Propagate the time.
    simulation_timer_.propagateTime(interactions_.totalRate());

    // Run the re-matching of the affected sites and their neighbours.
    const std::vector<int> & indices = \
        lattice_map_.supersetNeighbourIndices(process.affectedIndices(), process.range());

    matcher_.calculateMatching(interactions_,
                               configuration_,
                               sitesmap_,
                               lattice_map_,
                               indices);

    // Update the interactions' probability table.
    interactions_.updateProbabilityTable();

    // Update the interactions' process available sites.
    interactions_.updateProcessAvailableSites();
}
```
可以看到在第行上，进行更新的范围是当前选中并发生的事件，问题就在这里了，每个事件在定义的时候即使在后面的interaction类中进行了通配符的补全，但是范围是个数值并没有改变，所以这里如果正好选择的是一个小range的事件，而他周围有正好有个大range的事件包含了这个点，那么小range的更新就无法触及大range的事件，则大range的事件仍然认为某些实际不能发生的事件还是可以发生的，也就是上面图中的现象。
<br>
所以正确的修改应该是在每次求超集的时候都按照所有事件的最大范围来求，即
``` Cpp
// Run the re-matching of the affected sites and their neighbours.
const std::vector<int> & indices = \
    lattice_map_.supersetNeighbourIndices(process.affectedIndices(),
                                          interactions_.maxRange());
```

