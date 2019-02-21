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

# pyfi结构
pyfi的结构如下图所示：

```
pyfi
|_wind_helper: wind接口封装
|_visual:画图工具
|_common:公用函数
|_data_agent: 指标生成函数集
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

# Wind接口封装
pyfi中定义了WindHelper类，用于封装wind的python api函数
## mapper表
mapper表旨在用新的名称替代wind代码，方便提取。

```python
   mapper = Map({
    "ip_yoy": "M0000545",
    "ip_cyoy": "M0000011",
    "cpi": "M0000612",
    "cpif": "M0000616",
    "cpinf": "M0000613",
    "cpicore": "M0085932",
    "ppi": "M0001227",
    "cgpi": "M0001375",  # "企业商品交易价格指数"
    "USDNH即期汇率": "M0290205",
    "gz1y": "M1001646",
    "gz2y": "S0059745",
    "gz3y": "M1001648",
    "gz4y": "M0057946",
    "gz5y": "M1001650",
    "gz6y": "M0057947",
    "gz7y": "S0059748",
    "gz8y": "M1000165",
    "gz9y": "M1004678",
    "gz10y": "S0059749",
    "gz20y": "S0059751",
    "gz30y": "S0059752",
    "gk10y": "M1004271",
    "gk7y": "M1004269",
    "gk5y": "M1004267",
    "gk3y": "M1004265",
    "gk1y": "M1004687",
    "gc007": "204007.SH",
    "gc001": "204001.SH",
    "gc014": "204014.SH",
    "gc028": "204028.SH",
    "gc091": "204091.SH",
    "r007": "M1001795",
    "1yfr007": "M0048486",
    "5yfr007": "M0048490",
    "M2": "M0001385",
    "水泥价格指数": "S5914515",
    "CRB": "S0031505",
    "gz10yind": "CBA04501.CS",
    "玻璃综合指数": "S5907372",
    "螺纹钢": "S5707798",
    "fai_cyoy": "M0000273",
    "brent": "S0260035",  # 布伦特原油活跃期货合约结算价
})
```
整个mapper表用我们Map对象存储，支持据点索引。

```python
In[20]: mapper.水泥价格指数
Out[20]: 'S5914515'
```

## edb函数
WindHelper包含edb函数，主要对wind的edb函数进行了封装。

```python
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
- adjust：默认False,True则将月度数据对应的月末时间调整到最近的前一个交易日
- shift：默认0， 正数表示数据向后迁移的单位数，负数为向前
- fill：默认False，True则前一个月数据填充空缺数据

返回dataframe


案例：

```python
from pyfi import WindHelper as w
cpi = w.edb(codes=["cpi"], begin_date="2002-01-01", end_date="2019-01-01")
cpi.head()
```

# 回测功能

## 净价指数月度回测函数：monthly_backtest

```python
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
- `bk_code`: 默认`FIXEDINCOME`，即7-10年国债指数，可以自己添加指定基准的wind代码

返回结果：
回测函数返回一个自定义Map类型，可以直接通过句点索引返回信息中的各个数据，并且支持中文索引。
Map中的包含的信息：
- `score`:输入的打分
- `port`: 组合的净值曲线
- `pos`: 仓位
- `benchmark`: 基准的净值曲线
- `alpha`: 超额alpha
- `d_alpha`： 超额alpha的一阶变化
- `ytm`: 十年国债收益率月度数据
- `report`: 回测统计报告(建议调用pyfi.common.dprint函数打印report)
案例：

```python

from pyfi import monthly_backtest
from pyfi import WindHelper as w
cpi = w.edb(codes=["cpi"], begin_date="2002-01-01", end_date="2019-01-01")
cpi = cpi.iloc[:,0].apply(lambda x: 1 if x <2 else -1) # cpi的0阶逻辑
rlt = monthly_backtest(cpi)
from pyfi import dprint
dprint(rlt.report)
```

<img src="/pictures/pyfi_helper_document_backtest.PNG" style="display:block;margin:auto"/>

## 财富指数回测（含回购收益）

```python
monthly_backtest_with_repo(
                 score, 
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
- `init_weight`: 设定仓位基准区间，系统默认最低配置为0，最高配置为2倍的int_weight
- `bk_code`: 默认`FIXEDINCOME`，即7-10年国债指数，可以自己添加指定基准的wind代码

返回结果：
回测函数返回一个自定义Map类型，可以直接通过句点索引返回信息中的各个数据，并且支持中文索引。
Map中的包含的信息：
- `score`:输入的打分
- `port`: 组合的净值曲线
- `pos`: 仓位
- `benchmark`: 基准的净值曲线
- `alpha`: 超额alpha
- `ytm`: 十年国债收益率月度数据
- `report`: 回测统计报告(建议调用pyfi.common.dprint函数打印report)
案例：基于对CPI高点低点预测的有效性分析

```python

from pyfi import monthly_backtest_with_repo
from pyfi import WindHelper as w
from pyfi import logic
cpi = w.edb(codes=["cpi"], begin_date="2002-01-01", end_date="2019-01-01")
cpi = -logic.zo1(cpi.iloc[:,0], args=[6])
 # cpi的0阶逻辑(zo1),6个月的高点做空，6个月的低点做多
rlt = monthly_backtest_with_repo(cpi)
from pyfi import dprint
dprint(rlt.report)
```

<img src="/pictures/pyfi_helper_document_backtest_with_repo.PNG" style="display:block;margin:auto"/>

```
-  最大回撤:0.1658583853682296
-  夏普比率:0.735677258760662
-  超额夏普比率:0.03532301244161473
-  年化收益率:0.058541120038705596
-  基准年化收益率:0.03886420945842484
-  年均alpha:0.01967691058028076
-  alpha最大回撤:0.09773265095965566
-  信息比率:0.12626187165277092
-  calmar指标:0.3548018407116408
-  看多次数:43.0
-  看多准确率:0.7674418604651163
-  看空次数:66.0
-  看空准确率:0.5606060606060606
-  综合准确率:0.6422018348623854

```


# 指标生成集(待补充)

# 各类函数功能(待补充)
## 农历模块
农历模块可以帮助我们分析各个春节效应。

```python
def spring_list(begin_year, end_year)
```

输入参数：
- begin_year:起始年
- end_year: 结束年

返回：
- series类型，其中index为春节对应阳历日期，value（右侧）为农历日期 

案例:

```python
> spring_list(2002, 2018)
## 输出(左边是阳历，右边是农历)
2002-02-12   2002-01-01
2003-02-01   2003-01-01
2004-01-22   2004-01-01
2005-02-09   2005-01-01
2006-01-29   2006-01-01
2007-02-18   2007-01-01
2008-02-07   2008-01-01
2009-01-26   2009-01-01
2010-02-14   2010-01-01
2011-02-03   2011-01-01
2012-01-23   2012-01-01
2013-02-10   2013-01-01
2014-01-31   2014-01-01
2015-02-19   2015-01-01
2016-02-08   2016-01-01
2017-01-28   2017-01-01
2018-02-16   2018-01-01
```


## 收益率曲线boostrap模块
## 宏观序列处理模块

# 常用画图功能

pyfi提供必要的画图功能帮助大家快速查看时间序列的形态和比较。
## 单坐标line_graph画图函数：

```python
line_graph(series_list, 
		   title="multi lines graph", 
		   legend_list="", 
		   fig=None, 
		   ax=None, 
		   save=False)
```

案例:

```python
from pyfi import line_graph
gz10y = w.edb(codes=["gz10y"], begin_date="2002-01-01", end_date="2019-01-01").iloc[:,0]
line_graph([gz10y])
```

<img src="/pictures/pyfi_helper_document_line_graph.PNG" style="display:block;margin:auto"/>


## 双坐标double_line换图函数

```python
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

```python
from pyfi import double_lines
gz10y = w.edb(codes=["gz10y"], begin_date="2002-01-01", end_date="2019-01-01").iloc[:,0]
cpi = w.edb(codes=["cpi"], begin_date="2002-01-01", end_date="2019-01-01").iloc[:,0]
double_lines(gz10y, cpi)
```

<img src="/pictures/pyfi_helper_document_double_lines.PNG" style="display:block;margin:auto"/>

# 逻辑函数
## 单元逻辑函数
基于宏观量化方法论体系，我们初步提供了一些基础的0阶，1阶，2阶函数。
所有逻辑函数的输入为series，输出也是series
使用方法：

```python
from pyfi import logic
data_output = logic.zo1(data_input, args=[3])
# data_input和data_output都是series
```



### `zo1(data, args=[3])`
zero_order1,极值逻辑
> 判断当前观测点是否处于极值位置

输入参数：
- data：宏观序列Series
- args：参数组，单一参数，默认3，表示当前观测值是否为最近三个月的机制

返回：
- Series, 最高点返回1，最低点返回0

### `zo2(data, args=(36, 0))`
> zero_order2,标准化，并指定最低阈值，在正负阈值区间内为0

输入参数：
- data：宏观序列Series
- args：参数组，单一参数，args[0]默认36，表示观测区间，args[1]默认0，表示生效阈值 

返回：
- 布尔序列：最高点返回1，最低点返回0

### `fo1(data, args=[6])`
> first_order1:边际强度，表示序列在指定区间内的边际强度

输入参数：
- data：宏观序列Series
- args：参数组，单一参数，默认6，表示观测区间

返回：
- 离散序列，范围在[-args[0], args[0]]

### `fo2(data, args=[5])`
> first_order2: 趋势持续度，表示序列在区间内的持续度

输入参数：
- data：宏观序列Series
- args：参数组，单一参数，默认5，表示序列观测区间

返回：
- 离散序列，范围在[-args[0], args[0]]

### `fo3(data, args=[1, 12])`
> first_order3: 均线缺口

输入参数：
- data：宏观序列Series
- args：参数组，双参数，args[0]为短均线，args[1]为长均线

返回：
- 连续序列

### `so1(data, args=[5, 1])`
>second_order1: 边际强度变化的强度

输入参数：
- data：宏观序列Series
- args：参数组，单参数，args[0]表示观测区间，相当于fo1的一阶

返回：
- 离散序列，范围为[-args[0]+1, args[0]-1]


# 版本更新
## v0.1.19版本说明
- 修正了pandas的future warning，增加register_matplotlib_converters用于处理datetimeindex数据
- 增加monthly_backtests_with_repo回测函数，并增加bk_code
- 修正zo1单元逻辑函数，高点为1，低点为-1，其他为0

## v0.1.18版本说明
- 增加农历春节功能

## v0.1.16版本说明
- 本次修复了pandas更新造成的bug

## v0.1.15版本说明
- 本次更新重构了pyfi架构。
- 修改了部分单元逻辑函数。

# changelog
- 2019-02-20 update
- 2019-02-13 init

# author
- 大船
- 邮箱：wangluzhou@aliyun.com
