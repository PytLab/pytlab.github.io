---
title: 动力学模型使用Multiple-Thread
tags:
  - catalysis
  - chemistry
  - kinetic model
  - python
  - Kynetix
categories:
  - 学术
  - 代码作品
date: 2015-04-18 12:43:39
---

这两天想学习下优化代码，既然手底下有自己写完的动力学代码，那正好可以拿着练手。按我现在的认识程度，能对现在写好的动力学代码优化提升效率的方法除了改进算法外，单纯在编程方面一个是**使用多线程并行**，一个是**用c api把循环部分重写**。
把python多线程的一点点皮毛看了看就现学现卖了下。
把迭代的部分看了看，每步迭代里面都不是各自独立不相关的运算，真心不能给每个部分的运算分配子线程。在整个动力学模型里面最适合用多线程的地方就是作图了，因为作图中的阴影以及循环添加注释是可以分别分配子线程来进行的。于是我就把plotter里面画图的部分改了下:
分别添加了两个`threading.Thread`的子类`ShadowThread`，`NoteThread`用来实例化阴影和添加注释的子进程。

<!-- more -->

``` python
class ShadowThread(threading.Thread):
    "Sub-class of Thread class to create threads to plot shadows."
    def __init__(self, func, args):
        threading.Thread.__init__(self)
        self.func = func
        self.args = args

    def run(self):
        apply(self.func, self.args)

class NoteThread(threading.Thread):
    "Sub-class of Thread class to add notes to line."
    def __init__(self, func, args):
        threading.Thread.__init__(self)
        self.func = func
        self.args = args

    def run(self):
        apply(self.func, self.args)
```

然后就是在画图的代码中实例化thread子类，进行多线程。贴上添加注释部分的多线程代码：

``` python
    #use multiple thread to add notes
    note_threads = []
    nstates = len(tex_state_list)

    for pts, tex_states in zip(piece_points, tex_state_list):
        t = NoteThread(add_state_note, (pts, tex_states))
        note_threads.append(t)

    for i in xrange(nstates):
        note_threads[i].start()

    for i in xrange(nstates):
        note_threads[i].join()
```
大致过程就是循环实例化`thread`子类，然后收集到list中，在开始线程执行，最后依次用`join()`方法检验线程池中的线程是否结束，没有结束就阻塞直到线程结束，如果结束则跳转执行下一个线程的`join`函数。从而达到阻塞进程直到线程执行完毕的效果。
