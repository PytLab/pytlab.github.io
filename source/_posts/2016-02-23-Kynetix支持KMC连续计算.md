---
title: Kynetix支持KMC连续计算
date: 2016-02-23 19:26:10
tags:
  - Kynetix
  - python
  - catalysis
  - kMC
  - 学术
categories:
  - 学术
description: 由于KMC的计算经常会需要在已经算完的作业的基础上继续跑作业以达到TOF或者覆盖度稳定,因此使程序能够进行连续的作业计算。
---
关于连续计算这里其实是很早以前就加进来的功能，但是由于这里也算是一个比较重要而且比较复杂的更新，在这里记一下以免自己忘记。

这部分比较难处理的是关于连续作业的起始时间`start_time`和起始步数`start_step`的设定。然后再`KMCAnalysisPlugin`类（或者子类）中将新的KMC循环的time和step进行相应的**“平移处理”**。
这里我写了个装饰器来处理，对`KMCAnalysisPlugin`的`setup()`方法进行装饰，使其能够在进行on-the-fly分析的时候能够先把time和step处理完毕。
``` python
def reset_step_and_time(func):
    '''
    Decorator function to decorate analysis plugin methods.
    '''
    def wrapped_func(self, step, time, configuration):
        step = step + self.step_base
        time = time + self.time_base
        func(self, step, time, configuration)

    return wrapped_func

...

class KynetixPlugin(KMCAnalysisPlugin):
    ...

    @reset_step_and_time
    def setup(self, step, time, configuration):
        ...
```
<!-- more -->

关于如何获取起始的time和step，我是在`model.py`的`load()`函数来从文件中读取的，也就是说连续计算需要三个文件（均为程序自动生成，有`auto_`前缀）:
- `auto_coverages.py`, 存储上次作业的体系时间列表和KMC步数列表，若只是进行TOFAnalysis，则不需要此文件
- `auto_TOFs.py`, 存储上次作业的KMC步数和相应的TOF
- `auto_last_types.py`, 上次作业的最后一次表面的configuration

model.py中的读取部分代码不贴了，传送门：[<span class="fa fa-github"></span> Kynetix/model.py at master · PytLab/Kynetix](https://github.com/PytLab/Kynetix/blob/master/pynetics/model.py)

---
这里有两个变量容易混淆：`start_time`和`tof_start_time`，其中
- `start_time`就是上次作业停止时候体系的时间（体系到模拟结束时候体系真实推进的时间）
- `tof_start_time`就是**第一次**作业进行TOF统计的**时刻**，需要这个变量是为了在后续作业统计TOF的时候是使用的同一个起点，不然这连续作业对于统计TOF也没有任何意义了。
