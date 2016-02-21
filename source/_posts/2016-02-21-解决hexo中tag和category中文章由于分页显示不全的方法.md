---
title: 解决hexo中tag和category中文章由于分页显示不全的方法
date: 2016-02-21 14:12:58
tags:
  - hexo
categories:
  - 教程
---
今天忽然发现博客的标签(tag)以及目录(category)涉及到分页的话会显示不全，我发现之所以显示不全是和hexo中的`_config.yml`文件中的
``` 
per_page: 6
```
参数有关，`per_page`设成多少就只显示多少个post。

<!-- more -->

于是我看到在全局配置文件中还有几个参数我这边没有设：
```
index_generator:
  per_page: 10 ##首頁默認10篇文章標題 如果值爲0不分頁

archive_generator:
  per_page: 10 ##歸檔頁面默認10篇文章標題
  yearly: true  ##生成年視圖
  monthly: true ##生成月視圖

tag_generator:
  per_page: 10 ##標籤分類頁面默認10篇文章

category_generator:
   per_page: 10 ###分類頁面默認10篇文章


feed:
  type: atom ##feed類型 atom或者rss2
  path: atom.xml ##feed路徑
  limit: 20  ##feed文章最小數量


deploy:
  type: git ##部署類型 其他類型自行google之
  repo: <repository url> ##git倉庫地址
  branch: [branch] ##git 頁面分支
  message: [message] ##git message建議默認字段update 可以自定義
```

我就尝试把`tag_generator`和`category_generator`中的`per_page`设成了0：
```
tag_generator:
    per_page: 0

category_generator:
    per_page: 0
```

重新部署网站之后，OK! 问题解决！
