layout: post
title: KinLab - 基于Flask+Bootstrap实现的用于微观动力学计算的web应用
date: 2017-07-04 20:47:14
tags:
 - webapp
 - javascript
 - KinLab
 - kynetix
 - catalyst
 - chemistry
 - bootstrap
 - flask
 - python
 - javascript
 - jQuery
categories:
 - 代码作品
feature: /assets/images/blog_img/2017-07-04-使用Flask实现微观动力学计算的web应用/feature.png
toc: true
---
前两周花了些时间给自己的动力学程序Kynetix和KMCLibX写了个web界面方便别人使用，目前程序完成了计算微观动力学的部分，并部署在了腾讯云上http://123.206.225.154:5000/, 程序命名为KinLab (https://github.com/PytLab/kynetix-webapp).

目前KinLab主要包含四个部分:
1. 文件系统
2. 模型定义
3. 模型计算
生成结果报告

## 文件系统
文件系统这里我模仿jupyternotebook写的，因为着急实现主要的计算过程，这里一些常用的文件操作还没实现(后续有时间会加上)，正常可以在任何路径下创建或者打开一个作业设置界面。
<!-- more -->
![](/assets/images/blog_img/2017-07-04-使用Flask实现微观动力学计算的web应用/filestree.png)

同时在文件系统中可以查看文本文件的内容，若是非文本文件则会进行下载，其中代码高亮使用了rainbow插件显示。

![](/assets/images/blog_img/2017-07-04-使用Flask实现微观动力学计算的web应用/filecontent.png)

## 模型创建
在创建模型的时候，程序会判断当前路径下是否已经存在模型相关的文件，如果存在则会打开现有的模型，否则创建一个空的设置面板。

![](/assets/images/blog_img/2017-07-04-使用Flask实现微观动力学计算的web应用/openjob.png)

模型设置面板的界面如下:

![](/assets/images/blog_img/2017-07-04-使用Flask实现微观动力学计算的web应用/model-panel.png)

### 添加基元反应方程式
根据有无能垒的反应，可以自行选择，一下以包含能垒的反应式为例:

![](/assets/images/blog_img/2017-07-04-使用Flask实现微观动力学计算的web应用/choose-rxn.png)

创建基元反应的弹窗，为了能对基元反应进行检测，我特地把化学方程式解析的部分独立出来成一个模块rxn-parser.js(https://github.com/PytLab/rxn-parser.js)

![](/assets/images/blog_img/2017-07-04-使用Flask实现微观动力学计算的web应用/rxn-modal.png)

### 反应式存档
通过单击`save`可以将方程式信息保存到当前路径下

![](/assets/images/blog_img/2017-07-04-使用Flask实现微观动力学计算的web应用/rxn-save.png)

### 绘制energy-profile

当选中一个或者多个基元反应的时候，便可以绘制交互式的energy profile

![](/assets/images/blog_img/2017-07-04-使用Flask实现微观动力学计算的web应用/energy-profile.png)

其他相关功能在使用时可以进行尝试

### 模型参数

模型参数便是常用的求解微观动力学的参数，这里需要注意的是在添加好基元反应后，通过`Load`程序可以解析反应式中的物种并生成压强和总覆盖度表单。

![](/assets/images/blog_img/2017-07-04-使用Flask实现微观动力学计算的web应用/load-pressure.png)

## 作业计算
设置好模型后便可以`Run`了。程序会打开Running界面，并实时显示作业生成的日志

![](/assets/images/blog_img/2017-07-04-使用Flask实现微观动力学计算的web应用/running.png)

## 生成结果报告

报告会以图标和表格的方式显示:

其中包括ODE积分的轨迹图，稳态覆盖度，TOF以及可逆度。

![](/assets/images/blog_img/2017-07-04-使用Flask实现微观动力学计算的web应用/report.png)

