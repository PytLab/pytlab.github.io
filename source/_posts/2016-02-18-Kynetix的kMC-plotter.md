---
title: Kynetix的kmc_plotter
date: 2016-02-18 22:56:16
tags:
  - Kynetix
  - catalysis
  - kMC
  - Monte Carlo
  - chemistry
  - matplotlib
  - python
  - 学术
categories:
  - 学术
description: 简单介绍下用来绘制kMC模拟表面演化过程的绘图程序.
---
为了显示kMC模拟的表面动态演化过程,给Kynetix的plotters中新增了
- `kmc_plotter.py`, 其中的KMCPlotter是ModelShell的子类，实例化model对象的绘制工具。
- `grid_plot.py`， 提供绘制网格和圆圈的函数（适合小型网格，例如10x10、5x5）
- `scatter_plot.py`， 提供使用绘制散点图的方式绘制网格（适合大型的网格，例如200x200）
- `images2gif.py`， 负责将多个图片合并成动态gif的模块。

<!-- more -->

---
`kmc_plotter.py`中的`KMCPlotter`类中的`plot_traj`方法，是通过读取Kynetix运行自动生成的`auto_trajectory.py`文件的数据并使用网格或者散点的方式生成一系列的静态图片，也就是相应的轨迹图。

---
关于`images2gif.py`这个模块，是直接使用的现成的别人写好的模块([https://pypi.python.org/pypi/images2gif](https://pypi.python.org/pypi/images2gif)),但是一开始用的时候不知道为什么总是在合并的时候报错，去stackoverflow上面求助才知道新版本本来就会有这个问题，某个人提供的某个版本是可以的，于是我也就将这个版本直接复制过来放到了Kynetix中，以便能够稳定的绘图。

贴上绘制的效果(240x240):
![](assets/images/blog_img/2016-02-18-Kynetix的kMC-plotter/traj_movie.gif)
