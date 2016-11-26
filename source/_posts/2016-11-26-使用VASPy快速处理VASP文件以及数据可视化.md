---
title: 使用VASPy快速处理VASP文件以及数据可视化
date: 2016-11-26 16:15:26
tags:
 - VASPy
 - VASP
 - python
categories:
 - 学术
feature: /assets/images/features/python_logo.png
---

## 前言

本文为作者对其开源项目[VASPy](https://github.com/PytLab/VASPy)的说明文章。[VASPy](https://github.com/PytLab/VASPy)是一个纯Python编写的处理VASP文件数据以及进行数据快速可视化的库，基于OOP的思想提供了操作VASP文件的友好的接口，可以帮助使用者快速编写处理VASP相关文件的脚本，以提升效率。VASPy的项目仍处于起步阶段，希望大家可以都贡献出自己的力量使其壮大起来。

## VASP简介

对于广大做计算化学或者材料模拟的同学肯定听说过VASP的大名或者其科学研究与其息息相关。
VASP的全称是Vienna Ab-initio Simulation Package，是维也纳大学Hafner课题组开发的进行电子结构计算和量子力学-分子动力学模拟的软件包，目前是材料模拟和计算物质科学研究中最流行的商业软件之一。关于VASP的详细介绍可以参见其官方主页(http://www.vasp.at/)

<!-- more -->

## VASPy项目简介

VASPy的思想是将VASP相关的文件都视为可操作的对象，通过友好的接口对一个或者多个VASP对象进行快速的操作以提升工作效率。目前已兼容Python2 和 Python3。

- VASPy的GitHub地址：https://github.com/PytLab/VASPy
- VASPy的PyPI地址：https://pypi.python.org/pypi/vaspy/

![](/assets/images/blog_img/2016-11-26-使用VASPy快速处理VASP文件以及数据可视化/VASPy_PyPI.png)

## 使用说明

### 安装

VASPy库已上传至PyPI可以通过pip来进行安装:

```
$ pip install vaspy
```

从源码安装:
```
$ git clone git@github.com:PytLab/VASPy.git

$ cd vaspy

$ python setup.py install
```

### VASPy包的文件结构
```
VASPy/
├── LICENSE
├── MANIFEST
├── MANIFEST.in
├── README.rst
├── requirements.txt
├── scripts
│   ├── change_incar_parameters.py
│   ├── create_inputs.py
│   └── ...
├── setup.cfg
├── setup.py
├── tests
│   ├── incar_test.py
│   ├── __init__.py
│   ├── oszicar_test.py
│   ├── outcar_test.py
│   ├── testdata
│   │   ├── CONTCAR
│   │   ├── DOS_SUM
│   │   ├── ELFCAR
│   │   └── ...
│   └── ...
└── vaspy
    ├── __init__.py
    ├── iter.py
    ├── matstudio.py
    └── ...
```

### 文件操作举例

目前VASPy提供了操作INCAR、POSCAR、OUTCAR、XDATCAR、ELFCAR等的接口，这里对其中的部分进行简要的举例介绍。

#### 操作`INCAR`文件
`INCAR`是VASP做电子结构计算的参数设置文件，VASPy提供了`InCar`类可以方便获取INCAR文件的信息以及进行自定义的修改并生成新的INCAR文件。

``` python
In [1]: from vaspy.incar import InCar

In [2]: incar = InCar("INCAR")    # 创建InCar对象

In [3]: incar.IBRION              # 读取参数信息
Out[3]: '1'

In [4]: incar.ISIF
Out[4]: '2'

In [5]: incar.ISIF = 3             # 修改参数

In [6]: incar.tofile("INCAR_new")  # 生成新的INCAR文件
```

通过此类操作便可以快速写出批量修改INCAR文件的脚本，附上代码链接(https://github.com/PytLab/VASPy/blob/master/scripts/change_incar_parameters.py)

#### 操作`POSCAR`/`CONTCAR`/`XDATCAR`等含有结构坐标的文件

操作结构文件可以获取相应结构的信息，例如晶胞参数、晶胞体积等。
```python
In [7]: from vaspy.atomco import PosCar

In [8]: poscar = PosCar("POSCAR")

In [9]: poscar.bases
Out[9]: 
array([[  7.29321435,  -4.21073927,   0.        ],
       [  0.        ,   8.42147853,   0.        ],
       [ -0.        ,   0.        ,  16.87610843]])

In [10]: poscar.get_volume()
Out[10]: 1036.5246404472209

In [11]: poscar.data
Out[11]: 
array([[ 0.24466667,  0.224     ,  0.13581544],
       [ 0.02244444,  0.11288889,  0.27163089],
       [ 0.13355555,  0.00177777,  0.        ],
       ...                                   ])

```
同时结构坐标类中还提供了三维空间坐标转换接口，例如Cartisan坐标与Direct坐标的相互转换。

``` python
In [14]: poscar.cart2dir(self.bases, self.data)
Out[14]: ...

In [15]: poscar.dir2cart(self.bases, self.data)
Out[15]: ...
```

从XDATCAR中获取迭代的结构信息。

``` python
from vaspy.atomco import XdatCar
>>> xdatcar = XdatCar("XDATCAR")
>>> for step, data in xdatcar:
>>>     print(step)
>>>     print(xdatcar.dir2cart(xdatcar.bases, data))
```

#### 操作OUTCAR文件

`OUTCAR`是VASP最重要的输出文件，我们可以从中获取计算过程中基本上所有的信息。

获取迭代过程中原子的受力信息：
``` python
In [4]: from vaspy.iter import OutCar

In [5]: outcar = OutCar("OUTCAR_freq", poscar='POSCAR_freq')

In [9]: outcar.forces()  # 最近一次迭代中结构中原子在各个方向上的受力
Out[9]: 
([[2.79563, 0.85618, 1.19698],
  [4.47844, 0.86375, 4.78817],
  [2.37243, -0.5474, 3.59093],
  [3.91022, -0.54961, 7.26487],
  ...                        ])
```

如果要获取所以迭代步中的受力信息，需要使用OutCar提供的受力信息迭代器：

``` python
for forces in outcar.force_iterator:
    # Do something with forces tuple.
    ...
```

OutCar类对于含有频率计算的信息的文件会做频率收取操作，可以方便获取频率相关数据:

``` python
In [16]: outcar.freq_info
Out[16]: ('index', 'freq_type', 'THz', '2PiTHz', 'cm-1', 'meV', 'coordinates', 'deltas')

In [17]: outcar.freq_types
Out[17]: [['f', 'f', 'f'], ['f', 'f', 'f/i']]

In [19]: outcar.zpe
Out[19]: 0.1117761635

In [20]: for freq_info in outcar.freq_iterator:
    ...:     # Do something with frequency data
    ...:     ...

```

对于其他文件的操作这里就不进行一一介绍了。

### VASP数据可视化

#### 可视化分割后的DOS(态密度)数据

可视化的过程中可以选择进行d-band center的计算并显示。

``` python
In [1]: from vaspy.electro import DosX

In [2]: dos = DosX('DOS_SUM')

In [3]: dos.plotsum(0, (5, 10))
```
效果图:
![](/assets/images/blog_img/2016-11-26-使用VASPy快速处理VASP文件以及数据可视化/DOS.png)

#### `ELFCAR`/`CHGCAR`数据的可视化

电荷数据主要是通过对三维矩阵进行处理后进行绘制，可以选择surface以及二维map和标量场的显示模式。

```python
In [1]: from vaspy.electro import ElfCar

In [2]: elfcar = ElfCar("ELFCAR")

In [3]: elfcar.plot_contour()
```

![](/assets/images/blog_img/2016-11-26-使用VASPy快速处理VASP文件以及数据可视化/contour2d.png)

3D 等值线图, 这需要安装Mayavi模块来进行绘制。

```python
In [4]: elfcar.plot_contour3d()
```

![](/assets/images/blog_img/2016-11-26-使用VASPy快速处理VASP文件以及数据可视化/contour3d.png)


绘制标量场，同样需要Mayavi的支持。

```python
In [5]: elfcar.plot_field()
```

![](/assets/images/blog_img/2016-11-26-使用VASPy快速处理VASP文件以及数据可视化/field.png)

CHGCAR也是Fortran顺序的三维矩阵，绘制道理相同，因此可以用继承自ElfCar的ChgCar类来进行CHGCAR相关的绘制，例如差分电荷图。

```python
In [4]: from vaspy.electro import ChgCar

In [5]: chgcar = ChgCar("CHGCAR_diff")

In [6]: chgcar.plot_contour()
```

![](/assets/images/blog_img/2016-11-26-使用VASPy快速处理VASP文件以及数据可视化/contourf.png)

#### 操作MaterialStudio中的xsd以及xtd等文件的接口

VASPy还提供了一个方便将Material Studio中的`xsd`文件与VASP文件互通的接口，通过VASPy中的`XsdFile`和`XtdFile`类可以抽取文件中的晶格结构信息并结合VASP相关的类进行VASP文件的创建，同样可以方便的讲VASP的文件生成相应的用Material Studio可以显示的文件包括讲XDATCAR生成相应的`*.arc`和`*.xtd`来显示动画效果。

附上脚本的链接，此脚本就是利用VASPy的接口将Material Studio文件和VASP的文件进行相互转换。

- [由MaterialStudio的xsd文件生成VASP输入文件的脚本](https://github.com/PytLab/VASPy/blob/master/scripts/create_inputs.py)
- [由VASP的输出文件生成相应的MaterialStudio可以显示的xsd文件的脚本](https://github.com/PytLab/VASPy/blob/master/scripts/create_xsd.py)
- [由MaterialStudio的轨迹文件生成VASP进行NEB搜索过渡态的输入文件的脚本](https://github.com/PytLab/VASPy/blob/master/scripts/create_neb_inputs.py)

由VASP结果生成MaterialStudio的轨迹文件的动画效果图:

![](/assets/images/blog_img/2016-11-26-使用VASPy快速处理VASP文件以及数据可视化/sn2_my.gif)

## 结语
VASPy最初的想法是通过Python优雅简洁的特点将VASP的文件处理进行模块化，从而省去了重复写脚本的所花费的精力，使操作VASP文件像操作变量一样简单有效。
目前本项目都是在作者工作需要的基础上不断对其功能和接口进行完善，但仍只是冰山一角，希望做计算模拟使用VASP的Pythoner们能不断参与进来，使其更加出色和高效。

