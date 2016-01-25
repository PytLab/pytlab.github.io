---
title: VASPy一个面向对象的VASP文件处理库
id: 411
tags:
  - VASP
categories:
  - 代码作品
---

放假在家可以写写以前的想法，把vasp中的文件写成python对象，这样既系统有方便写脚本。就在github上开了个repo，目前写了几个文件的class,其中包含处理MaterialStudio的xsd类xml文件的类。不断更新,欢迎使用vasp的小伙伴们star,fork贡献出自己的一份力。
库链接：[<ins datetime="2015-08-12T06:17:43+00:00">Processing VASP files with Python - VASPy</ins>](https://github.com/PytLab/VASPy)

<!-- more -->

---
软件包介绍
---

### An **object-oriented** VASP file processing library.

Make it **easier** to process VASP files.

处理VASP文件从未如此 **灵活** **简单**


### 命令行处理DOS文件使用举例：

    #处理分割好的DOS文件
    >>> from vaspy.electro import DosX
    >>> a = DosX('DOS1')
    >>> b = DosX('DOS8')
    
    #分波态密度合并
    >>> c = a
    >>> c.reset_data()              # 初始化DOS数据
    >>> for i in xrange(1, 10):
    >>>    c += DosX('DOS'+str(i))  # 循环合并DOS数据
    >>> ...
    >>> c.data                      # 以float矩阵显示合并后的数据
                                    # 可直接进行计算等操作
    >>> c.tofile()                  # 生成新的合并后的DOS文件
    
    #绘图
    >>> c.plotsum(0, (5, 10))       # 绘制d轨道pDOS图
    
#### 绘制结果:

![](pDOS.png)

处理ELFCAR举例:

    >>> from vaspy.electro import ElfCar
    >>> a = ElfCar() 
    >>> a.plot_contour()   # 绘制等值线图
    >>> a.plot_mcontour()  # 使用mlab绘制等值线图(需安装Mayavi)
    >>> a.plot_contour3d() # 绘制3d等值线图
    >>> a.plot_field()     # 绘制标量场

#### 绘制结果:

![](contour2d.png)

![](contours.png)

3D 等值线图

![](contour3d.png)

scalar field

![](field.png)

charge difference(use ChgCar class)

![](contourf.png)

操作XDATCAR举例

    >>> from vaspy.atomco import XdatCar
    >>> xdatcar = XdatCar()
    >>> # 输出xdatcar相应Cartesian坐标
    >>> for step, data in xdatcar:
    >>>     print step
    >>>     print xdatcar.dir2cart(xdatcar.bases, data)
    >>> # 可直接运行script/中脚本生成相应.arc文件用于MaterialStudio显示动画
    >>> python xdatcar_to_arc.py

动画实例

![](sn2_my.gif)

**使用者可以编写自己的脚本来批处理VASP文件**

