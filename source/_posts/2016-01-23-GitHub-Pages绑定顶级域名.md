---
title: GitHub Pages绑定顶级域名的方法
date: 2016-01-23 18:49:43
tags:
  - Hexo
  - GitHub Pages
  - 域名
  - GitHub
categories:
  - 教程
image: 
  feature: 
excerpt: test
comments: true
---

刚刚把自己买的域名[ipytlab.com](http://ipytlab.com)与GitHub Pages进行绑定，现在访问[ipytlab.com](http://ipytlab.com)以及[pytlab.github.io](http://pytlab.github.io)均能访问我的博客啦。

在github的help页面有介绍如何绑定域名 - [About custom domains for GitHub Pages sites](https://help.github.com/articles/about-custom-domains-for-github-pages-sites/)

下面简单写一下我将Hexo + Github Pages绑定顶级域名的方法：
1. 在自己网站项目repo的根目录添加CNAME，里面的内容为域名不要http以及www等前缀，只需写入域名本身，例如
    ```
    ipytlab.com
    ```
    {% alert warning %}
    如果是直接在GitHub网页上添加文件的话，会遇到一个问题就是在通过`hexo g -d`之后hexo会把根目录下的CNAME文件删除。
    {% endalert %}
    所以要把CNAME文件添加到`/source`目录下，这样`hexo g -d`之后hexo会自动把CNAME复制到`/puclic`目录下然后将`/public`路径下的内容进行复制并push到远程`master`分支的根目录下。

    <!-- more -->

2. 添加DNS Service记录
    在[DNSPod](https://www.dnspod.cn)注册帐号然后添加域名设置两个A记录，分别是@和www，ip地址填
    ```
    192.30.252.153
    ```
    如下图：
    ![](DNSPod.png)

3. 设置域名的DNS
    在相应域名的DNS Service中添加上图中间的两条记录：
    `f1g1ns1.dnspod.net`和`f1g1ns1.dnspod.net`

4. 稍等解析生效后就可以通过在浏览器中输入自己的域名来访问GitHub Pages博客啦! 如下图，
    ![](homepage.png)
