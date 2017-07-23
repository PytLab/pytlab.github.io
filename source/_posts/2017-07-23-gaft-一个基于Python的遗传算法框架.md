layout: post
title: GAFT-一个使用Python实现的遗传算法框架
date: 2017-07-23 15:27:28
tags:
 - python
 - GeneticAlgorithm
categories:
 - 代码作品
feature: /assets/images/blog_img/2017-07-23-gaft-一个基于Python的遗传算法框架/surface.png
toc: true
---

## 前言
最近需要用到遗传算法来优化一些东西，最初是打算直接基于某些算法实现一个简单的函数来优化，但是感觉单纯写个非通用的函数运行后期改进算子或者别人使用起来都会带来困难，同时遗传算法基本概念和运行流程相对固定，改进也一般通过编码机制，选择策略，交叉变异算子以及参数设计等方面，对于算法的整体结构并没有大的影响。这样对于遗传算法来说，就非常适合写个相对固定的框架然后给算子、参数等留出空间以便对新算法进行测试和改进。于是就动手写了个遗传算法的小框架gaft，本文对此框架进行一些介绍并分别以一个一维搜索和二维搜索为例子对使用方法进行了介绍。

<!-- more -->

GitHub: https://github.com/PytLab/gaft
PyPI: https://pypi.python.org/pypi/gaft

目前框架只是完成了最初的版本，比较简陋，内置了几个基本的常用算子，使用者可以根据接口规则实现自定义的算子并放入框架中运行。我自己也会根据自己的需求后续添加更多的改进算子，同时改进框架使其更加**通用**.

![](/assets/images/blog_img/2017-07-23-gaft-一个基于Python的遗传算法框架/title.png)

## 正文

### 遗传算法介绍
这里我对遗传算法的基本概念进行简要的介绍，并阐述gaft的设计原则。

简单而言，遗传算法使用群体搜索技术，将种群代表一组问题的可行解，通过对当前种群施加选择，交叉，变异等一些列遗传操作来产生新一代的种群，并逐步是种群进化到包含近似全局最优解的状态。下面我将遗传学和遗传算法相关术语的对应关系总结一下:

#### 术语

|遗传学术语       |遗传算法术语    |
|:----------------|:---------------|
|群体             |可行解集        |
|个体             |可行解          |
|染色体           |可行解的编码    |
|基因             |可行解编码的分量|
|基因形式         |遗传编码        |
|适应度           |评价函数值      |
|选择             |选择操作        |
|交叉             |交叉操作        |
|变异             |变异操作        |

#### 算法特点
1. 以决策变量的编码作为运算对象，使得优化过程借鉴生物学中的概念成为可能
2. 直接以目标函数作为搜索信息，确定搜索方向很范围，属于无导数优化
3. 同时使用多个搜索点的搜索信息，算是一种隐含的并行性
4. 是一种基于概率的搜索技术
5. 具有自组织，自适应和自学习等特性

#### 算法流程

![](/assets/images/blog_img/2017-07-23-gaft-一个基于Python的遗传算法框架/flowchart.png)

#### gaft 设计原则
由于遗传算法的流程相对固定，我们优化算法基本上也是在流程整体框架下对编码机制，算子，参数等进行修改，因此在写框架的时候，我便想把那些固定的遗传算子，适应度函数写成接口，并使用元类、装饰器等方式实现对接口的限制和优化，这样便可以方便后续自定义算符和适应度函数定制。最后将各个部分组合到一起组成一个engine然后根据算法流程运行遗传算法对目标进行优化.

这样我们便脱离每次都要写遗传算法流程的繁琐，每次只需要像写插件一样实现自己的算子和适应度函数便可以将其放入gaft开始对算法进行测试或者对目标函数进行优化了。

### GAFT文件结构

此部分我对自己实现的框架的整体结构进行下介绍.
```
.
├── LICENSE
├── MANIFEST.in
├── README.rst
├── examples
│   ├── ex01
│   └── ex02
├── gaft
│   ├── __init__.py
│   ├── __pycache__
│   ├── analysis
│   ├── components
│   ├── engine.py
│   ├── operators
│   └── plugin_interfaces
├── setup.cfg
├── setup.py
└── tests
    ├── flip_bit_mutation_test.py
        ├── gaft_test.py
        ├── individual_test.py
        ├── population_test.py
        ├── roulette_wheel_selection_test.py
        └── uniform_crossover_test.py
```
目前的文件结果如上所示，
- `/gaft/components`中定义了内置的个体和种群类型，提供了两种不同的遗传编码方式:二进制编码和实数编码。
- `/gaft/plugin_interfaces`中是插件接口定义，所有的算子定义以及on-the-fly分析的接口规则都在里面，使用者可以根据此来编写自己的插件并放入到engine中。
- `/gaft/operators`里面是内置遗传算子，他们也是遵循`/gaft/plugin_interfaces`中的规则进行编写，可以作为编写算子的例子。其中算子我目前内置了roulette wheel选择算子，uniform 交叉算子和flipbit变异算子，使用者可以直接使用内置算子来使用gaft对自己的问题进行优化。
- `/gaft/analysis`里面是内置的on-the-fly分析插件，他可以在遗传算法迭代的过程中对迭代过程中的变量进行分析，例如我在里面内置了控制台日志信息输出，以及迭代适应度值的保存等插件方便对进化曲线作图。
- `/gaft/engine`便是遗传算法的流程控制模块了，他将所有的之前定义的各个部分组合到一起使用遗传算法流程进行优化迭代。

### 使用GAFT

下面我就以两个函数作为例子来使用GAFT对目标函数进行优化.

#### 一维搜索

首先我们先对一个简单的具有多个局部极值的函数进行优化，我们来使用内置的算子求函数
$$
f(x) = x + 10sin(5x) + 7cos(4x)
$$
的极大值，x的取值范围为$[0, 10]$

1. **先导入需要的模块**
    ``` python
    from math import sin, cos

    # 导入种群和内置算子相关类
    from gaft import GAEngine
    from gaft.components import GAIndividual
    from gaft.components import GAPopulation
    from gaft.operators import RouletteWheelSelection
    from gaft.operators import UniformCrossover
    from gaft.operators import FlipBitMutation

    # 用于编写分析插件的接口类
    from gaft.plugin_interfaces.analysis import OnTheFlyAnalysis

    # 内置的存档适应度函数的分析类
    from gaft.analysis.fitness_store import FitnessStoreAnalysis

    # 我们将用两种方式将分析插件注册到遗传算法引擎中
    ```

2. **创建引擎**

    ``` python
    # 定义种群
    indv_template = GAIndividual(ranges=[(0, 10)], encoding='binary', eps=0.001)
    population = GAPopulation(indv_template=indv_template, size=50)

    # 创建遗传算子
    selection = RouletteWheelSelection()
    crossover = UniformCrossover(pc=0.8, pe=0.5)
    mutation = FlipBitMutation(pm=0.1)

    # 创建遗传算法引擎, 分析插件和适应度函数可以以参数的形式传入引擎中
    engine = GAEngine(population=population, selection=selection,
                      crossover=crossover, mutation=mutation,
                      analysis=[FitnessStoreAnalysis])
    ```

3. **自定义适应度函数**

    可以通过修饰符的方式将，适应度函数注册到引擎中。

    ``` python
    @engine.fitness_register
    def fitness(indv):
        x, = indv.variants
        return x + 10*sin(5*x) + 7*cos(4*x)
    ```

4. **自定义on-the-fly分析插件**

    也可以通过修饰符在定义的时候直接将插件注册到引擎中

    ``` python
    @engine.analysis_register
    class ConsoleOutputAnalysis(OnTheFlyAnalysis):
        interval = 1

        def register_step(self, ng, population, engine):
            best_indv = population.best_indv(engine.fitness)
            msg = 'Generation: {}, best fitness: {:.3f}'.format(ng, engine.fitness(best_indv))
            engine.logger.info(msg)

        def finalize(self, population, engine):
            best_indv = population.best_indv(engine.fitness)
            x = best_indv.variants
            y = engine.fitness(best_indv)
            msg = 'Optimal solution: ({}, {})'.format(x, y)
            engine.logger.info(msg)
    ```

5. **Ok, 开始跑(优化)吧！**

    我们这里跑100代种群.

    ``` python
    if '__main__' == __name__:
        # Run the GA engine.
        engine.run(ng=100)
    ```

内置的分析插件会在每步迭代中记录得到的每一代的最优个体，并生成数据保存。

绘制一下函数本身的曲线和我们使用遗传算法得到的进化曲线:

![](/assets/images/blog_img/2017-07-23-gaft-一个基于Python的遗传算法框架/envolution_curve_1d.png)

优化过程动画:

![](/assets/images/blog_img/2017-07-23-gaft-一个基于Python的遗传算法框架/animation.gif)

#### 二维搜索

下面我们使用GAFT内置算子来搜索同样具有多个极值点的二元函数
$$
f(x) = ysin(2\\pi x) + xcos(2\\pi y)
$$

的最大值，x, y 的范围为 $[-2, 2]$.

这里我们就不自定义分析插件了，直接使用内置的分析类，并在构造引擎时直接传入.

``` python
'''
Find the global maximum for binary function: f(x) = y*sim(2*pi*x) + x*cos(2*pi*y)
'''

from math import sin, cos, pi

from gaft import GAEngine
from gaft.components import GAIndividual
from gaft.components import GAPopulation
from gaft.operators import RouletteWheelSelection
from gaft.operators import UniformCrossover
from gaft.operators import FlipBitMutation

# Built-in best fitness analysis.
from gaft.analysis.fitness_store import FitnessStoreAnalysis
from gaft.analysis.console_output import ConsoleOutputAnalysis

# Define population.
indv_template = GAIndividual(ranges=[(-2, 2), (-2, 2)], encoding='binary', eps=0.001)
population = GAPopulation(indv_template=indv_template, size=50)

# Create genetic operators.
selection = RouletteWheelSelection()
crossover = UniformCrossover(pc=0.8, pe=0.5)
mutation = FlipBitMutation(pm=0.1)

# Create genetic algorithm engine.
# Here we pass all built-in analysis to engine constructor.
engine = GAEngine(population=population, selection=selection,
                  crossover=crossover, mutation=mutation,
                  analysis=[ConsoleOutputAnalysis, FitnessStoreAnalysis])

# Define fitness function.
@engine.fitness_register
def fitness(indv):
    x, y = indv.variants
    return y*sin(2*pi*x) + x*cos(2*pi*y)

if '__main__' == __name__:
    engine.run(ng=100)
```

进化曲线:
![](/assets/images/blog_img/2017-07-23-gaft-一个基于Python的遗传算法框架/envolution_curve.png)

二维函数面:
![](/assets/images/blog_img/2017-07-23-gaft-一个基于Python的遗传算法框架/surface.png)

搜索过程动画:
![](/assets/images/blog_img/2017-07-23-gaft-一个基于Python的遗传算法框架/surface_animation.gif)

可见目前内置的基本算子都能很好的找到例子中函数的最优点。

## 总结
本文主要介绍了本人开发的一个用于遗传算法做优化计算的Python框架，框架内置了遗传算法中常用的组件，包括不同编码方式的个体，种群，以及遗传算子等。同时框架还提供了自定义遗传算子和分析插件的接口，能够方便快速搭建遗传算法流程并用于算法测试。

目前框架仅仅处于初步阶段，后续会在自己使用的过程中逐步完善更多的内置算子，是框架更加通用。本文中的两个优化例子均能在GitHub上找到源码(https://github.com/PytLab/gaft/tree/master/examples)

目前的计划：1. 添加更多的内置算子; 2. 给内置算子和组件添加C++ backend; 3. 并行化

## 参考
- 《智能优化算法及其MATLAB实例》
- 《MATLAB最优化计算》

