---
title: Windows下py2与py3共存的方法
date: 2016-04-30 15:43:39
tags:
  - python
  - windows
categories:
  - 教程
description: "最近打算转移阵地至python3，但是有有老代码是python2的，于是就像在电脑上装两个版本的python和相应的ipython。下面记录一下处理方法。"
feature: /assets/images/features/ipython.jpg
toc: true
---

现在的情况是我已经在我的系统中安装了python2.7以及ipython和其他我经常用的库。现在我需要在安装python3.5的解释器以及相应版本的ipython。

### 1. 安装python3.5.1

直接去官网下载后安装到默认路径下。我安装的路径为：

```
C:\Users\xy\AppData\Local\Programs\Python\Python35-32
```

在这路径下可以看到python3.5的所有，为了与之前的python2.7分开，我将python重命名为`python3.exe`

<!-- more -->
![](/assets/images/blog_img/2016-04-30-Windows下py2与py3共存的方法/py3dir.png)

### 2. 添加至环境变量
为方便在shell中进入解释器，将python3.5的安装路径以及下面的Scripts路径添加至环境变量。
![](/assets/images/blog_img/2016-04-30-Windows下py2与py3共存的方法/env_var.png)
添加scripts是为了安装ipython后方便启动。

### 3. 安装pip3
之前的pip是安装python2相应的库的，为了能给py3方便安装PyPI的库，需要重新安装pip3。
方法就是去下载[get-pip.py](https://bootstrap.pypa.io/get-pip.py)，然后使用python3去执行，pip3便会安装在Script路径下。
![](/assets/images/blog_img/2016-04-30-Windows下py2与py3共存的方法/pip3.png)

### 4. 安装ipython
有了pip3，就可以直接在shell中使用`pip install`来安装ipython了。
```
$ pip install ipython
```
相应版本的ipython就会安装在Scripts路径下
![](/assets/images/blog_img/2016-04-30-Windows下py2与py3共存的方法/ipython3.png)

### 安装完毕
这样我们就可以在shell中即可以执行py3也可以执行py2了。

#### 进入python2的ipython
![](/assets/images/blog_img/2016-04-30-Windows下py2与py3共存的方法/ipython2shell.png)

#### 进入python3的ipython
![](/assets/images/blog_img/2016-04-30-Windows下py2与py3共存的方法/ipython3shell.png)
