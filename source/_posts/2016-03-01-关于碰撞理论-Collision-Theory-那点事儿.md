---
title: 关于碰撞理论(Collision Theory)那点事儿
date: 2015-11-01 19:40:29
tags:
  - catalysis
  - chemistry
  - 学术
  - kinetics
categories:
  - 学习小结
description: "针对碰撞理论总结一下，免得自己过时间久了又去翻书"
---
这里只总结关于使用碰撞理论进行表面碰撞（吸附）速率的推导。
先上个图吧，理解这里全靠这张图。
![](surface_collision.png)
<!-- more -->
碰撞速率就是单位时间内单位面积分子碰撞到表面的次数。如图给定一个表面其面积为$A$, 若分子能够碰撞到表面则一定有垂直于表面的速率$v\_{x}$。
所有分子的速率分布服从Boltzmann分布$f(v\_{x})$。
若垂直于表面的速率为$v\_{x}$的分子能在在给定时间$\\Delta t$内能够撞到表面的话，则该分子距离表面的距离一定小于$v\_{x}\\Delta t$。
换一个角度理解就是，若在垂直于表面且体积为$V = v\_{x}\\Delta t A$的立方体之内的分子才能够撞到表面。
那就好办了，也就是说把这个体积求出来乘上分子的密度就得到时间$\\Delta t$内撞到表面积为$A$的表面的分子数了。那速率也就可以得到了。于是有，
$$
r\_{coll-surf} = \\frac{1}{\\Delta t A }\\int\_{0}^{\\infty} f(v\_{x})V(v\_{x}) \\rho dv\_{x}
$$
其中$\\int\_{0}^{\\infty} f(v\_{x})V(v\_{x}) \\rho dv\_{x}$就是能够碰撞到表面的分子总数
能够撞到表面的“体积”   $V=v\_{x}\\Delta t A$也是速率的函数因此要放在里面一起积分。
<br>
于是碰撞速率就可推导出：
$$
r\_{coll-surf} = \\frac{1}{\\Delta t A }\\int\_{0}^{\\infty} f(v\_{x})V(v\_{x}) \\rho dv\_{x} = \\rho \\int\_{0}^{\\infty} v\_{x} f(v\_{x}) dv\_{x} \\\
= \\frac{p}{k\_{B}T} \\int\_{0}^{\\infty}  v\_{x} \\sqrt{\\frac{m}{2\\pi k\_{B} T}  } e^{- \\frac{mv\_{x}^{2}}{2k\_{B}T}} dv\_{x} \\\
= \\frac{p}{k\_{B}T} \\bar{v\_{x}} = \\frac{p}{k\_{B}T} \\sqrt{\\frac{k\_{B}T}{2\\pi m}} 
= \\frac{p}{\\sqrt{2 \\pi m k\_{B} T}}
$$

{% alert danger %}
这里得到的是单位时间单位面积的碰撞次数，也就是单位为$m^{-2} \bullet s^{-1}$
{% endalert%}
