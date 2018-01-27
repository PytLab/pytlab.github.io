---
layout: post
title: 实现属于自己的TensorFlow(二) - 梯度计算与反向传播
date: 2018-01-25 20:49:55
tags:
 - MachineLearning
 - TensorFlow
 - SimpleFlow
 - NeuralNetwork
 - DeepLearning
 - Backpropagation
categories:
 - 代码作品
feature: /assets/images/blog_img/2018-01-24-实现属于自己的TensorFlow-一-计算图与前向传播/feature.png
toc: true
---

## 前言

[上一篇](http://pytlab.github.io/2018/01/24/%E5%AE%9E%E7%8E%B0%E5%B1%9E%E4%BA%8E%E8%87%AA%E5%B7%B1%E7%9A%84TensorFlow-%E4%B8%80-%E8%AE%A1%E7%AE%97%E5%9B%BE%E4%B8%8E%E5%89%8D%E5%90%91%E4%BC%A0%E6%92%AD/)中介绍了计算图以及前向传播的实现，本文中将主要介绍对于模型优化非常重要的反向传播算法以及反向传播算法中梯度计算的实现。因为在计算梯度的时候需要涉及到矩阵梯度的计算，本文针对几种常用操作的梯度计算和实现进行了较为详细的介绍。如有错误欢迎指出。

首先先简单总结一下, 实现反向传播过程主要就是完成两个任务:

1. 实现不同操作输出对输入的梯度计算
2. 实现根据链式法则计算损失函数对不同节点的梯度计算

再附上SimpleFlow的代码地址: https://github.com/PytLab/simpleflow

<!-- more -->

## 正文

### 反向传播

对于我们构建的模型进行优化通常需要两步：1.求损失函数针对变量的梯度；2.根据梯度信息进行参数优化(例如梯度下降). 那么该如何使用我们构建的计算图来计算损失函数对于图中其他节点的梯度呢？通过**链式法则**。我们还是通过上篇中的表达式$Loss(x, y, z) = z(x + y)$对应的计算图来说明:

![](/assets/images/blog_img/2018-01-25-实现属于自己的TensorFlow-二-梯度计算与反向传播/forward.png)

我们把上面的操作节点使用字母进行标记，可以将每个操作看成一个函数，接受一个或两个输入有一个或者多个输出, 则上面的表达式
$$Loss(x, y, z) = z(x+y)$$
可以写成
$$Loss(x, y, z) = g(z, f(x, y))$$

那么根据链式法则我们可以得到$Loss$对$x$的导数为:
$$
\\frac{\\partial Loss}{\\partial x} = \\frac{\\partial Loss}{\\partial g}\\frac{\\partial g}{\\partial f} \\frac{\\partial f}{\\partial x}
$$

假设图中的节点已经计算出了自己的输出值，我们把节点的输出值放到节点里面如下:

![](/assets/images/blog_img/2018-01-25-实现属于自己的TensorFlow-二-梯度计算与反向传播/forward_with_values.png)

然后再把链式法则的式子每一项一次计算，在图中也就是从后向前进行计算:

1. $\\frac{\\partial Loss}{\\partial g} = 1$
    ![](/assets/images/blog_img/2018-01-25-实现属于自己的TensorFlow-二-梯度计算与反向传播/bp_0.png)

2. $\\frac{\\partial g}{\\partial f} = z = 6$ (当然也可以计算出$\\frac{\\partial g}{\\partial z} = x + y = 5$). 进而求出$\\frac{\\partial Loss}{\\partial f} = \\frac{\\partial Loss}{\\partial g}\\frac{\\partial g}{\\partial f} = 1 \\times z = 6$
    ![](/assets/images/blog_img/2018-01-25-实现属于自己的TensorFlow-二-梯度计算与反向传播/bp_1.png)

3. $\\frac{\\partial f}{\\partial x} = 1$ (同时也可以算出$\\frac{\\partial f}{\\partial y} = 1$). 进而求出$\\frac{\\partial Loss}{\\partial x} = \\frac{\\partial Loss}{\\partial g}\\frac{\\partial g}{\\partial f}\\frac{\\partial f}{\\partial x} = 1 \\times z \\times 1 = 6$
    ![](/assets/images/blog_img/2018-01-25-实现属于自己的TensorFlow-二-梯度计算与反向传播/bp_2.png)

这样从后向前逐级计算通过链式法则就可以计算出与损失值对其相关节点的梯度了。因此我们下一步要做的就是给定某个**损失函数**节点并计算它对于**某一节点**的**梯度**计算。

下面在看一个不同的计算图:

![](/assets/images/blog_img/2018-01-25-实现属于自己的TensorFlow-二-梯度计算与反向传播/bp_multiout.png)

这里的$x$节点有将输出到两个不同的节点中，此时我们需要计算所有从$g$到$x$的路径然后按照上面单挑路径的链式法则计算方法计算每条路径的梯度值，最终再将不同路径的梯度求和即可。因此$Loss$对$x$的梯度为:
$$
\\frac{\\partial Loss}{\\partial x} = \\frac{\\partial g}{\\partial f}\\frac{\\partial f}{\\partial h}\\frac{\\partial h}{\\partial x} + \\frac{\\partial g}{\\partial f}\\frac{\\partial f}{\\partial l}\\frac{\\partial l}{\\partial x}
$$

### 梯度计算

通过上面对反向传播的介绍我们已经知道损失值对某个节点的梯度是怎么求的(具体的实现方法在下一部分说明)，下面就是如何求取针对某个节点上的梯度了，只要每个节点上的梯度计算出来沿着路径反方向不断乘下去就会得到你想要的节点的梯度了。本部分就介绍如何求损失值对具体某个节点的梯度值。

本部分我们就是干这么一个事，首先我们先画个节点:

![](/assets/images/blog_img/2018-01-25-实现属于自己的TensorFlow-二-梯度计算与反向传播/single_node.png)

$f$节点可以看成一个函数$z = f(x, y)$， 我们需要做的就是求$\\frac{\\partial f(x, y)}{\\partial x}$和$\\frac{\\partial f(x, y)}{\\partial y}$.

#### 平方运算的梯度计算

我们先用一个平方运算（之所以不用求和和乘积/矩阵乘积来做例子，因为这里面涉及到矩阵求导维度的处理，会在稍后进行总结, 而平方运算并不会涉及到维度的变化比较简单):

``` python
class Square(Operation):
    ''' Square operation. '''
    # ...
    def compute_gradient(self, grad=None):
        ''' Compute the gradient for square operation wrt input value.

        :param grad: The gradient of other operation wrt the square output.
        :type grad: ndarray.
        '''
        input_value = self.input_nodes[0].output_value

        if grad is None:
            grad = np.ones_like(self.output_value)

        return grad*np.multiply(2.0, input_value)
```

其中`grad`为损失值对`Square`输出的梯度值，也就是上图中的$\\frac{\\partial Loss}{\\partial z}$的值, 它的`shape`一定与`Square`的输出值的`shape`**一致**。

#### 神经网络反向传播的矩阵梯度计算

矩阵梯度的计算是实现反向传播算法重要的一部分, 但是在实现神经网络反向传播的矩阵求导与许多公式列表上罗列出来的还是有差别的。

**矩阵/向量求导**

首先先看下矩阵的求导，其实矩阵的求导本质上就是目标矩阵中的元素对变量矩阵中的元素求偏导，至于求导后的导数矩阵的形状大都也都是为了形式上的美观方便求导之后的继续使用。所以不必被那些复杂的矩阵求导形式迷惑了双眼。这里上传了一份[矩阵求导公式法则的列表PDF版本](../../../../assets/files/matrix_rules.pdf)，可以一步一步通过（行/列）向量对标量求导再到（行/列）向量对（行/列）向量求导再到矩阵对矩阵的求导逐渐扩展。

例如标量$y$对矩阵$X = \\left[ \\begin{matrix}x\_{11} & x\_{12} \\\ x\_{21} & x\_{22} \\end{matrix} \\right]$求导, 我们就对标量$y$对于$X$的所有元素求偏导，最终得到一个导数矩阵，矩阵形状同$X$相同:

$$
\\frac{\\mathrm{d}y}{\\mathrm{d}X} = \\left[ \\begin{matrix}
\\frac{\\partial y}{\\partial x\_{11}} & \\frac{\\partial y}{\\partial x\_{12}} \\\
\\frac{\\partial y}{\\partial x\_{21}} & \\frac{\\partial y}{\\partial x\_{22}}
\\end{matrix} \\right]
$$

**神经网络反向传播中的矩阵求导**

之所以把矩阵求导分成两部分，是因为在实现矩阵求导的时候发现做反向传播的时候的矩阵求导与矩阵求导公式的形式上还是有区别的。所谓的**区别**就是，***我们在神经网络进行矩阵求导的时候其实是Loss(损失)函数对节点中的矩阵进行求导，而损失函数是标量，那每次我们对计算图中的每个节点计算梯度的时候其实是计算的标量(损失值)对矩阵(节点输出值)的求导***. 也就是说在进行反向传播的时候我们用的只是矩阵求导中的一种，即**标量对矩阵的求导**, 也就是上面举的例子的形式。再进一步其实就是损失函数对矩阵中每个元素进行求偏导的过程，通俗的讲就是计算图中矩阵中的每个元素对损失值的一个影响程度。因此这样计算出来的导数矩阵的形状与变量的形状一定是**一致**的。

直观上理解就是**计算图中对向量/矩阵求导的时候计算的是矩阵中的元素对损失值影响程度的大小，其形状与矩阵形状相同。**

#### 求和操作的梯度计算

现在我们以求和操作的梯度计算为例说明反向传播过程中矩阵求导的实现方法。

对于求和操作: $C = A + b$, 其中$A = \\left[ \\begin{matrix} a\_{11} & a\_{12} \\\ a\_{21} & a\_{22} \\end{matrix} \\right]$, $b = b\_{0}$, 则$C = \\left[ \\begin{matrix}  a\_{11} + b\_0 & a\_{12} + b\_0 \\\ a\_{21} + b\_0 & a\_{22} + b\_0 \\end{matrix} \\right]$, 损失值$L$对$C$梯度矩阵为 $G = \\left[ \\begin{matrix} \\frac{\\partial L}{\\partial c\_{11}} & \\frac{\\partial L}{\\partial c\_{12}} \\\ \\frac{\\partial L}{\\partial c\_{21}} & \\frac{\\partial L}{\\partial c\_{22}} \\end{matrix} \\right]$

下面我们计算$\\frac{\\partial L}{\\partial b}$, 根据我们之前说的这个梯度的维度(形状)应该与$b$相同，也就是一个标量,那么具体要怎么计算呢？我们分成两部分来处理：

1. 先计算对于$C = A + B$， $\\frac{\\partial L}{\\partial B}$的梯度值，其中$B = \\left[ \\begin{matrix} b\_0 & b\_0 \\\ b\_0 & b\_0 \\end{matrix} \\right]$是通过对$b$进行广播操作得到的
$$
\\frac{\\partial L}{\\partial B} =
\\left[ \\begin{matrix} 
\\frac{\\partial L}{c\_{11}} \\frac{\\partial c\_{11}}{\\partial b\_0} &
\\frac{\\partial L}{c\_{12}} \\frac{\\partial c\_{12}}{\\partial b\_0} \\\
\\frac{\\partial L}{c\_{21}} \\frac{\\partial c\_{21}}{\\partial b\_0} &
\\frac{\\partial L}{c\_{22}} \\frac{\\partial c\_{22}}{\\partial b\_0} \\\
\\end{matrix} \\right] =
\\left[ \\begin{matrix} 
\\frac{\\partial L}{c\_{11}} \\times 1 &
\\frac{\\partial L}{c\_{12}} \\times 1 \\\
\\frac{\\partial L}{c\_{21}} \\times 1 &
\\frac{\\partial L}{c\_{22}} \\times 1 \\\
\\end{matrix} \\right] = \\frac{\\partial L}{\\partial C} = G
$$

2. 计算$L$对$b$的梯度$\\frac{\\partial L}{\\partial b}$。因为$B$是对$b$的一次广播操作，虽然是用的是矩阵的形式，本质上是将$b$复制了4份然后再进行操作的，因此将$\\frac{\\partial L}{\\partial B}$中的每个元素进行累加就是$\\frac{\\partial L}{\\partial b}$的值了。

    则梯度的值为:
$$
\\frac{\\partial L}{\\partial b} = \\sum\_{i=1}^{2} \\sum\_{j=1}^{2} \\frac{\\partial L}{\\partial c\_{ij}}
$$
    针对此例$b$是一个标量，使用矩阵表示的话可以表示成:
$$
\\frac{\\partial L}{\\partial b} = 
\\left[ \\begin{matrix} 1 & 1 \\end{matrix} \\right]
G
\\left[ \\begin{matrix} 1 \\\ 1 \\end{matrix} \\right]
$$

    若$b$是一个长度为2的列向量，型如$\\left[ \\begin{matrix} b\_0 \\\ b\_0\\end{matrix} \\right]$ 则需要将$G$中的每一列进行相加得到与$b$形状相同的梯度向量:
$$
\\frac{\\partial L}{\\partial b} = 
\\left[ \\begin{matrix}
\\frac{\\partial L}{\\partial c\_{11}} + \\frac{\\partial L}{\\partial c\_{12}} \\\
\\frac{\\partial L}{\\partial c\_{21}} + \\frac{\\partial L}{\\partial c\_{22}}
\\end{matrix} \\right]
$$

下面是求和操作梯度计算的Python实现:
``` python
class Add(object):
    # ...

    def compute_gradient(self, grad=None):
        ''' Compute the gradients for this operation wrt input values.

        :param grad: The gradient of other operation wrt the addition output.
        :type grad: number or a ndarray, default value is 1.0.
        '''
        x, y = [node.output_value for node in self.input_nodes]

        if grad is None:
            grad = np.ones_like(self.output_value)

        grad_wrt_x = grad
        while np.ndim(grad_wrt_x) > len(np.shape(x)):
            grad_wrt_x = np.sum(grad_wrt_x, axis=0)
        for axis, size in enumerate(np.shape(x)):
            if size == 1:
                grad_wrt_x = np.sum(grad_wrt_x, axis=axis, keepdims=True)

        grad_wrt_y = grad
        while np.ndim(grad_wrt_y) > len(np.shape(y)):
            grad_wrt_y = np.sum(grad_wrt_y, axis=0)
        for axis, size in enumerate(np.shape(y)):
            if size == 1:
                grad_wrt_y = np.sum(grad_wrt_y, axis=axis, keepdims=True)

        return [grad_wrt_x, grad_wrt_y]
```

其中`grad`参数就是上面公式中的$G$它的shape应该与该节点的输出值(`output_value`的形状一直)。

#### 矩阵乘梯度的计算

这部分主要介绍如何在反向传播求梯度中运用**维度分析**来帮助我们快速获取梯度。先上一个矩阵乘操作的例子:

$$ C = AB $$

其中， $C$是$M \\times K$的矩阵, $A$是$M \\times N$的矩阵, $B$是$N \\times K$的矩阵。

损失值$L$对$C$的梯度为

$$G = \\frac{\\partial L}{\\partial C}$$

其形状与矩阵$C$相同同为$M \\times K$

通过维度分析可以通过我们标量求导的知识再稍微对矩阵的形状进行处理(左乘，右乘，转置)来**凑**出正确的梯度。当然如果需要分析每个元素的导数也是可以的，可以参考这篇[神经网络中利用矩阵进行反向传播运算的实质](https://zhuanlan.zhihu.com/p/25496760), 下面我们主要使用维度分析来快速计算反向传播中矩阵乘节点中矩阵对矩阵的导数。

若我们想求$\\frac{\\partial L}{\\partial B}$, 根据标量计算的链式法则应该有:

$$
\\frac{\\partial L}{\\partial B} = \\frac{\\partial L}{\\partial C} \\frac{\\partial C}{\\partial A}
$$
    
根据向量已知的$\\frac{\\partial L}{\\partial C}$的形状为$M \\times K$（与$C$形状相同）, $\\frac{\\partial L}{\\partial B}$的形状为$N \\times K$(与$B$形状相同), 因此$\\frac{\\partial C}{\\partial B}$ 应该是一个$N \\times M$的矩阵，而且我们上面乘积的式子写反了，把顺序调换一下就是

$$
\\frac{\\partial L}{\\partial B} = \\frac{\\partial C}{\\partial B} \\frac{\\partial L}{\\partial C}
$$

根据我们在标量求导的规则里，$C$对于$B$求导应该是$A$, 但是$A$是一个$M \\times N$的矩阵而我们现在需要一个$N \\times M$的矩阵，那么就将$A$转置一下呗，于是就得到:

$$
\\frac{\\partial L}{\\partial B} = \\frac{\\partial C}{\\partial B} \\frac{\\partial L}{\\partial C} = A^{T}G
$$

同理也可以通过维度分析得到$L$对$A$的梯度为$GB^{T}$

下面是矩阵乘操作梯度计算的Python实现:
``` python
class MatMul(Operation):
    # ...

    def compute_gradient(self, grad=None):
        ''' Compute and return the gradient for matrix multiplication.

        :param grad: The gradient of other operation wrt the matmul output.
        :type grad: number or a ndarray, default value is 1.0.
        '''
        # Get input values.
        x, y = [node.output_value for node in self.input_nodes]

        # Default gradient wrt the matmul output.
        if grad is None:
            grad = np.ones_like(self.output_value)

        # Gradients wrt inputs.
        dfdx = np.dot(grad, np.transpose(y))
        dfdy = np.dot(np.transpose(x), grad)

        return [dfdx, dfdy]
```

#### 其他操作的梯度计算

这里就不一一介绍了其他操作的梯度计算了，类似的我们根据维度分析以及理解反向传播里矩阵梯度其实就是标量求梯度放到了矩阵的规则里的一种变形的本质，其他梯度也可以推导并实现出来了。

在simpleflow里目前实现了求和，乘法，矩阵乘法，平方，Sigmoid，Reduce Sum以及Log等操作的梯度实现，可以参考:https://github.com/PytLab/simpleflow/blob/master/simpleflow/operations.py

## 总结

本文介绍了通过计算图的反向传播快速计算梯度的原理以及每个节点相应梯度的计算和实现，有了每个节点的梯度计算我们就可以通过实现反向传播算法来实现损失函数对所有节点的梯度计算了，下一篇中将会总结通过广度优先搜索实现图中节点梯度的计算以及梯度下降优化器的实现。

## 参考

- http://www.deepideas.net/deep-learning-from-scratch-iv-gradient-descent-and-backpropagation/
- https://zhuanlan.zhihu.com/p/25496760
- http://blog.csdn.net/magic_anthony/article/details/77531552
- https://www.zhihu.com/question/47024992/answer/103962301

