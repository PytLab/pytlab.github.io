---
title: git push的时候用ssh放弃https
tags:
  - Git
  - GitHub
date: 2015-08-29 09:23:35
---

这几天改以前的代码，每次git push到github的时候总是提醒我输入我输入github的用户名和密码，很是麻烦。
于是找到了原因，之前我的代码是通过https clone到本地的，如果一开始是通过git ssh克隆就不会出现这种问题。如果已经通过https克隆，不用着急可以修改。

在你的版本库.git中打开config文件会有如下一段代码:
```
[remote "origin"]  
fetch = + refs/heads/*:refs/remotes/origin/*  
url = https://username@github.com/username/projectname.git 
```
url改成相应的
```
[remote "origin"]  
fetch = + refs/heads/*:refs/remotes/origin/*  
url = git@github.com:username/projectname.git 
```
这样就可以通过ssh公钥的方式认证进行git push啦
