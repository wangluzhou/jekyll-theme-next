---
title: pyfi说明文档
date: 2018-08-10 10:04:58
categories:
- 技术
tags:
- Python
description: pyfi_helper的帮助文档
---

> pyfi_helper(以下简称pyfi)主要为宏观或者固收从业人员在使用Python做数据分析的过程中提供便捷的功能。
pyfi设计原则是基于宏观量化方法论体系，即将逻辑分为要素，单元逻辑，并行逻辑，串行逻辑进行分类，并将逻辑输出标签化，实现“精准抽象，模糊正确”的研究原则，并在此基础开发一些功能函数辅助大家进行投资研究。

# pyfi的结构

# pyfi的安装

首先你需要安装好以下几个包：
- wind的python api
- talib
然后再下载pyfi包：
```
pip install pyfi_helper
```
更新pyfi包：

```
pip install --upgrade pyfi_helper
```

# 调用wind数据

## edb函数


# 回测功能

回测函数默认基于打分机制，且打分必须符合中心为0，左右对称的内在机制。
默认正分为看多，负分为看空。

# 常用画图功能

pyfi提供必要的画图功能帮助大家快速查看时间序列的形态和比较。

# 逻辑函数一览
基于宏观量化方法论体系，我们初步提供了一些基础的0阶，1阶，2阶函数。

# changelog
- 2019-02-13 init

# author
- 大船