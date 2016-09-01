---
title: 'CentOS:Network is unreachable'
date: 2016-09-01 18:42:34
tags:
 - linux
 - CentOS
 - VMWare
categories:
 - 我的日常
feature: /assets/images/features/Linux-Inside.png
---
自己的虚拟机上的CentOS连接不上网了，ping外网的ip会显示
``` bash
Connetion: Network is unreachable
```
一开始以为是虚拟机的问题，把虚拟机的网卡卸载又重新安装也还是没用，这时候便是linux本身的问题了。

网络重启也会失败：
``` bash
# service network restart
Restarting network (via systemctl): Job for network.service failed. See *systemctl status network.service* and *journalctl -xn* for details.
[FAILED]
```
<!-- more -->

于是我就看了下日志到底是哪里出错
![](/assets/images/blog_img/2016-09-01-CentOS-Network-is-unreachable/error.png)
可见是网卡的物理地址冲突，于是就需要去相应网卡的配置文件去修改物理地址。
首先先看目前网卡的真正物理地址是什么:
``` bash
$ ip addr
```
![](/assets/images/blog_img/2016-09-01-CentOS-Network-is-unreachable/ipaddr.png)

然后修改配置文件:
``` bash
# vi /etc/sysconfig/network-scripts/ifcfg-eno16777736
```
将`HWADRR`的值修改或者添加成当前网卡地址.

最后通过
``` bash
# service network restart
```
重启网卡，或者通过
``` bash
# ifup eth0
```
启动网卡（该命令会检查配置文件）.

这样就恢复啦，pip, yum, 可以正常使用了.

