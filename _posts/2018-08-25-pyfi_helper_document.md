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

# v0.1.13版本说明
本次更新重构了pyfi架构。


# pyfi结构：
pyfi的结构如下图所示：

```
pyfi
|_wind_helper: wind接口封装
|_visual:画图工具
|_common:公用函数
|_data_agent: 指标计算函数包
|_backtest: 回测框架
|_function: 各类函数功能
|_logic: 逻辑函数库
```
# pyfi的安装

首先你需要安装好以下几个包：
- wind的python api
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
pyfi的edb函数主要对wind进行了封装，并命名了一些常用的变量，方便提取。
```Python
WindHelper.edb(codes, 
			   begin_date, 
			   end_date, 
			   options="fill=perious", 
			   adjust=False, 
			   shift=0, 
			   fill=False)
```
输入参数说明：
- codes: 代码的列表
- begin_date； 开始时间， str或者datetime类型
- end_date：结束时间，str或者datetime类型
- options：默认"fill=perious"
- adjust：默认False 
- shift：默认0 
- fill：默认False

返回dataframe


案例：
```Python
from pyfi import WindHelper as w
cpi = w.edb(codes=["cpi"], begin_date="2002-01-01", end_date="2019-01-01")

```

# 回测功能

## 净价指数月度回测函数：monthly_backtest
```Python
monthly_backtest(score, 
				 pattern=1, 
				 title="Monthly backtest with net value index", 
				 weight_bounds=[-1, 1],
				 bk_code=FIXEDINCOME)
```
输入参数:
- score:打分序列，回测函数默认基于打分机制，且打分必须符合中心为0，左右对称。
默认正分为看多，负分为看空。
- `pattern`: 数字表示打分的最大值
- `title`: 给回测图设定标题
- `weight_bounds`: 给仓位设定区间
- `bk_code`: 默认`FIXEDINCOME`，即7-10年国债指数，可以自己添加对应记住的wind代码

返回结果：
回测函数返回一个自定义Map类型，可以直接通过句点索引返回信息中的各个数据，并且支持中文索引。
Map中的包含的信息：
- `score`:输入的打分
- `port`: 组合的净值曲线
- `pos`: 仓位
- `benchmark`: 基准
- `alpha`: 超额alpha
- `d_alpha`： 超额alpha的一阶变化
- `ytm`: 十年国债收益率月度数据
- `report`: 回测统计报告(建议调用pyfi.common.dprint函数打印report)
example：
```
from pyfi import monthly_backtest

from pyfi import WindHelper as w

cpi = w.edb(codes=["cpi"], begin_date="2002-01-01", end_date="2019-01-01")

cpi = cpi.iloc[:,0].apply(lambda x: 1 if x <2 else -1) # cpi的0阶逻辑
rlt = monthly_backtest(cpi)
```

# 常用画图功能
pyfi提供必要的画图功能帮助大家快速查看时间序列的形态和比较。
## 单坐标line_graph画图函数：
```Python
line_graph(series_list, 
		   title="multi lines graph", 
		   legend_list="", 
		   fig=None, 
		   ax=None, 
		   save=False)
```
案例:
```Python
from pyfi import line_graph
gz10y = w.edb(codes=["gz10y"], begin_date="2002-01-01", end_date="2019-01-01").iloc[:,0]
line_graph([gz10y])
```

## 双坐标double_line换图函数:
```Python
 double_lines(series1,
             series2,
             lgd1="series1",
             lgd2="series2",
             title="double axis graph",
             colors=("grey", "red"),
             figname="",
             fig=None,
             ax=None):
```
案例：
```Python
from pyfi import double_lines
gz10y = w.edb(codes=["gz10y"], begin_date="2002-01-01", end_date="2019-01-01").iloc[:,0]
cpi = w.edb(codes=["cpi"], begin_date="2002-01-01", end_date="2019-01-01").iloc[:,0]
double_lines(gz10y, cpi)
```

# 逻辑函数（待完善）
基于宏观量化方法论体系，我们初步提供了一些基础的0阶，1阶，2阶函数。
所有逻辑函数的输入为series，输出也是series
- zero_order1: 极值逻辑
- zero_order2: 标准化
- zero_order3: 滚动分位
- first_order1: 边际强度
- first_order2: 趋势持续度
- first_order3:均线缺口
- second_order1: 边际强度变化的强度


# changelog
- 2019-02-20 update
- 2019-02-13 init

# author
- 大船
- 邮箱：wangluzhou@aliyun.com
