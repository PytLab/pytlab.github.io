---
title: 动力学模型添加QuasiEquilibriumSolver
tags:
  - catalysis
  - chemistry
  - kinetic model
  - kinetics
  - python

categories:
  - 学术
mathjax: true
date: 2015-05-17 22:12:34
---

在用sympy符号运算将动力学模型重新实现一遍以后，出了能进行latex输出等其他好处外，最初使用符号运算的目的是符号求解敏感度 $X\_{TRC}$的值，不过由于完全用solve暴力解析求解稳态覆盖度，稍微复杂点的机理就会计算很久。于是我打算用平衡态近似来进行求解，并且写完平衡态solver后不仅仅是求$X\_{TRC}$，单独用QuasiEquilibriumSolver求解动力学模型也是可以的。

为了避免直接用sympy.solve()函数直接求解，分析了平衡态近似的求解方法后，自己用python先对求解过程进行了一下"预处理"，这样从一定程度上减小了对solve()依赖。由于这部分还是有点麻烦的，所以写在这里记录下来，方便自己以及其他人学习和改进。

由于平衡态近似求解的方法还是比较单一的，大概过程就是在决定Rate Determining Step后，通过联立其他基元反应的平衡条件，也就是$\\prod\_{j} \\theta\_{i,j}^{-} \\prod\_{j} p\_{i,j}^{-} =K\_{i}\\prod\_{j} \\theta\_{i,j}^{+}\\prod\_{j} p\_{i,j}^{+}$用$\\theta^{\*}$将其他的$\\theta\_{i}$表示出来，在回代到归一化条件中$\\sum\_{i}{\\theta\_{i}} = 1$,将$\\theta^{*}$的表达式求出来，然后分别回代求出不同中间态的覆盖度。
计算方法很简单，但是这里涉及到对sympy.Symbol对象的引用和substitution以及后面基元反应的特殊情况，所以写起来略复杂。下面以甲酸裂解的一条路径的基元反应为例说明一下。
