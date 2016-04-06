---
title: KMCLib中的CustomRateProcess
date: 2016-03-21 09:52:10
tags:
  - Cpp
  - KMCLib
  - catalysis
  - chemistry
  - Monte Carlo
categories:
  - 学术
description: "CustomRateProcess是Process的派生类，<br>主要表示不同位置上速率不同的反应"
feature: /assets/images/features/rate_table.png
toc: true
---
此类是`Process`的派生类，所以在`Process`类中的成员变量为`protected`，而且很多成员函数都为虚函数。

### 成员数据
``` Cpp
// 由于不同site上面的速率不同，因此此vector存放对应sites_的速率
std::vector<double> site_rates_;

// 递增的速率列表
std::vector<double> incremental_rate_table_;
```
<!-- more -->

具体可以用下图表示：
![](assets/images/blog_img/2016-03-21-KMCLib中的CustomRateProcess/rate_table.png)

### 成员函数
其他成员函数的作用与Process类似，只是由于每个位点的速率不同所采用的求和和抽取的方法稍有差别，这里就不做详解了。
``` Cpp
// 根据site_rates_生成incremental_rate_table_
void CustomRateProcess::updateRateTable()
{
    // Resize and update the incremental rate table.
    incremental_rate_table_.resize(site_rates_.size());
    double previous = 0.0;
    for (size_t i = 0; i < site_rates_.size(); ++i)
    {
        incremental_rate_table_[i] = previous + site_rates_[i];
        previous = incremental_rate_table_[i];
    }

    // DONE
}
```
