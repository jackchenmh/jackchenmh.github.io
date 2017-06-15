---
title: 移动app数据分析框架（一）
tags:
    - pandas
    - python
    - data analysis
    - DataFrame
    - mobile app
    - DAU
    - DNU
    - numpy random choice
    - numpy random randint
    - Timedelta
    - groupby
    - Series.unique
    - date_range
    - resample
    - plot
    - hist
    -
categories:
    -
date: 2017-04-13
updated: 2017-04-13
layout: post
comments: true
---

Pandas是数据分析利器。它是基于NumPy 的一种工具，纳入了大量库和一些标准的数据模型，提供了高效操作大型数据集所需的工具。pandas提供了大量能使我们快速便捷地处理数据的函数和方法，比如数据透视表，布尔索引，统计函数，绘图，时间序列分析等。此外，它的性能也十分突出，参考[这篇文章](http://www.justi-blog.com/archives/1357), 加载近1亿条纪录也仅需要260秒左右。在这样的一个数据库中，查看数据表中的空值并进行True/False填充也只需要28.7秒。这个速度实际上比数据库要快很多。因此，使用Pandas来对大多数移动app（用户人数在千万级）进行数据分析，无论从容量还是性能上看都是足够的，而在代码实现时间上则会占到不少便宜。

本篇介绍与用户相关的数据的分析，包括用户激活（DNU），用户活跃(DAU), 用户登录次数的计算方法。在结尾再次回顾了本文使用的pandas相关函数。

<!--more-->

本框架使用服务端数据跟踪，从日志文件中（通过配置的filter)抓取事件信息，将信息保存到pandas DataFrame对象之中（backed by disk file)，并且提供Web界面以呈现数据。

为实现简便起见，Web界面数据呈现采用flask。数据通过DataFrame的to_html（）方法来生成页面。绘图使用matplotlib。

这个方案没有考虑数据访问控制，只适合中小团队，或者项目早期用于快速验证。

# 数据的生成与分析
## 用户相关数据
与用户相关的数据有用户注册数据（注册时间及要求填写的其它个人资料）、用户登录时间、用户在线时长等原始数据，并以此计算以下数据：
* 实时激活数据（一小时，当天累积，DNU，周、月）
* 活跃用户数据（日、周、月）
* 留存数据（+1,+2，+3,...)
* 流失率

移动端app在用户首次登录时，一般都能收集到IMEI, IMSI，IP，平台信息（ios或者android），设备名（phone6或者iphone6s），应用信息（版本号等），并根据IMEI来生成新的用户ID。这个用户ID会是在今后分析用户行为时，串起用户活动的关键所在。服务器在输出日志时，应该时刻注意输出这一字段。在此后的正式注册时，还会收集手机号，其它IM号，广告tracking ID（如果是iphone）。

在首次登录以后，用户可能流失，也可能继续登录。每次登录时，服务器都应该记录下用户登录的时间，以及IP,设备信息等。

建议使用一个pandas.DataFrame对象来保存用户基本资料，可以叫做user_profile。从应用发行以来，所有用户的档案数据都保存在这里，所以一些易变的数据，比如每次登录时间不要保存在这张表里。实际操作中建议根据用户量的大小，再分表保存这些数据。比如按用户注册时间分年、甚至分季度来保存用户档案。这张表的索引应该是user_id。表格格式如下：
user id  |  register_time | -me  | gendre |  VIP |  phone number |  ...
--|---|---|---|---|---|--
100086  | 2000/01/01 18:36:23.009  | N/A  | male  |  1 |  17002549030 |   |  

用户的每次登录都要记录在日志中，包括用户登录的时间，以及IP,设备信息等，这些信息是每次登录都可能发生变化的数据。这些信息可用来计算实时激活、用户活跃、留存和流失率数据。这些数据应该以如下格式存为临时表：
timestamp  |user id   |IP   |device   |...  
--|---|---|---|--
2000/01/01 18:00:23.123  | 10000  |  192.168.1.3 | sumgsun  |  ...
* 激活数据（DNU）

激活数据一般有如下重要用途：
1. 用来追踪广告投放的效果。这个追踪链条中，包括了广告展示量、点击量、下载量、下载完成量、激活量，甚至还可以跟踪到最终付费用户量。
2. 是留存率、新用户付费率等其它重要数据的分母。

在日志爬虫运行时，就会提取用户首次注册信息，保存到user_profile表格中。通过user_id和register_time这两列数据，我们就可以得到各种时间段的激活数据。要统计每半小时（甚至时每分钟），每小时，每天(DNU)，每周和每月的激活数据，只需要使用pandas序列分析的resample功能就可以了。我们来演示一下。
```python
# to calculate activation stats
import pandas as pd
import numpy as np

index = pd.date_range('1/1/2000', periods=5000, freq='S')

# to mimic randon login time, unit is second
index = np.random.choice(index, 1000)
series = pd.Series(range(1000), index=index)

# generate hourly activation stats
# hourly is another Series, with login time as index, count number as values
# 代码用'H'来重采样到小时单位。对于天单位和周单位，使用'D'和'W'。对于月和年，有起始
# 和结束一说（即是使用2000/01/01为标签，还是使用2000/01/31为标签）。对前者，使用
# 'MS'，对后者使用'M'。对年起始使用'AS'(这样年标签类似2017/01/01)，否则使用'A'（这样年标签类似2017/12/31)
hourly = series.resample('H').count()

# to access a specific hour using index, sub:
print("Access series using index, subscription:")
print(hourly['2000-01-01 00:00:00'], hourly['2000-01-01 01:00:00'],"\n", hourly[0], hourly[1])

# you can also treat hourly as a dict like this:
print("Treat serices as dict:")
for key in hourly.index:
    print(key, hourly[key])
```
这段代码首先模拟了每5秒随机注册一个用户的情况，然后按小时进行重采样，并统计每小时的激活人数，输出如下：
```bash
Access series using index, subscription:
62 64
62 64
Treat serices as dict:
2000-01-01 00:00:00 62
2000-01-01 01:00:00 64
2000-01-01 02:00:00 59
2000-01-01 03:00:00 68
2000-01-01 04:00:00 48
2000-01-01 05:00:00 58
2000-01-01 06:00:00 50
2000-01-01 07:00:00 61
2000-01-01 08:00:00 69
2000-01-01 09:00:00 60
2000-01-01 10:00:00 58
2000-01-01 11:00:00 63
2000-01-01 12:00:00 65
2000-01-01 13:00:00 63
2000-01-01 14:00:00 47
2000-01-01 15:00:00 58
2000-01-01 16:00:00 47
```
这样，要取得任意时间间隔（小时，天，周，月）的激活数据，只需要做重采样就可以了，所有复杂的计算，都由pandas完成了。
* 用户日登录统计
这些数据在某种程度上反映了用户喜爱app的程度。那些每日登录次数最多的用户是这款应用的核心用户，需要重点关注。通过将这些数据与竞品相比较，可以知道自己应用的健康度。

原始数据的来源跟激活数据一样，只不过我们需要处理所有的登录记录，而不是象激活数据那样，只关心首次登录。

下面的代码统计了每日登录总数login_times_perday, 用户平均登录次数login_times_average_perday，最大登录次数login_peruser_perday_max。
除了输出上述各项统计数据外，还将每天用户登录次数以直方图的形式展示出来，从中可以看出，绝大多数人只登录一次。
```python
# to caculate login times
%matplotlib inline

import pandas as pd
import numpy as np
import matplotlib

login_time = pd.date_range('1/1/2000', periods=5000, freq='55S')
# mimic random login records
login_time = np.random.choice(login_time, 1000)

ids = np.random.randint(1, 200, 1000)
series = pd.Series(ids, index=login_time)

# login times per day in total, should add up to 1000
login_times_perday = series.resample('D').count()

# unique login ids in total per day. each group is passed in as x
unique_ids_perday = series.groupby(pd.DatetimeIndex(login_time).date).apply(lambda x : len(x.unique()))
print("\nunique login users per day\n",unique_ids_perday)

# print out daily user login times
login_times_average_perday =  login_times_perday/unique_ids_perday
print("\ndaily average user login:\n", login_times_average_perday)

#
login_peruser_perday_max = series.groupby(pd.DatetimeIndex(login_time).date)
print("\nmax login times:\n", login_peruser_perday_max.apply(lambda x : x.value_counts().values[0]))
login_peruser_perday_max.value_counts().hist()
```
输出如下：
```python
login times per day:
 2000-01-01    326
2000-01-02    307
2000-01-03    313
2000-01-04     54
Freq: D, dtype: int64

unique login users per day
 2000-01-01    155
2000-01-02    161
2000-01-03    160
2000-01-04     47
dtype: int64

daily average user login:
 2000-01-01    2.103226
2000-01-02    1.906832
2000-01-03    1.956250
2000-01-04    1.148936
Freq: D, dtype: float64

max login times:
 2000-01-01    5
2000-01-02    5
2000-01-03    5
2000-01-04    3
dtype: int64
```
![](移动app数据分析框架\9ab26cb94bd90b91ee983426814541e8.png)

* 活跃用户数据（DAU）

一般来说我们需要每日、每月的活跃用户数据，分别记作DAU和MAU。MAU/DAU的值反映了一个应用的健康程度，这个值越逼近1，则应用健康度越高。它的解释是，你的用户几乎每天都使用你的应用。

在前面一节中已经给出了DAU的计算方法，即unique login users per day。注意本文的方法中，并未包括当天注册的新用户，而实际上大多数统计方法中，会将这一数据也纳入。

因此，我们需要从user_profile和user_login两张表中，取出每天的unique_login_users_per_day，再相加。

这次我们的代码使用两个DataFrame数据结构--这基本上就是日志爬虫的输出形态--而不是Series来完成这个统计。
```python
%matplotlib inline

import pandas as pd
import numpy as np
import matplotlib

register_time = pd.date_range('1/1/2000', periods=7, freq='D')
# mimic random login records
register_time = np.random.choice(register_time, 200)

ids = np.random.randint(0, 200, 200)

# create user profile table. Each user can register only once
user_profile = pd.DataFrame({'user_id': ids}, index = register_time)
user_profile['register_time'] = register_time

# create user login table. User login time must be later than register time, and  many records may exist
login_ids = np.random.choice(ids, 1000)
login_timestamp = lambda id: user_profile['register_time'][id] + pd.Timedelta(np.random.randint(0, 500000), unit='s')
login_time = [login_timestamp(id) for id in login_ids]

user_login = pd.DataFrame({'time_stamp': login_time, 'user_id': login_ids}, index = login_time)

# unique login ids in total per day found in user_login table:
from_user_login = user_login.groupby(pd.DatetimeIndex(login_time).date).apply(lambda x : len(x['user_id'].unique()))
print("\nunique login users per day\n",from_user_login)

# unique login ids in total per day found in user_profile table:
from_user_register = user_profile.groupby(pd.DatetimeIndex(register_time).date).apply(lambda x : len(x['user_id'].unique()))
print("\n daily register users:\n",from_user_register)

# print out DAU
dau = from_user_login + from_user_register
print("\nDAU is:\n", dau)

dau.plot()
```
输出结果如下，注意部分结果出现-N，这是因为from_user_register中的数据行比from_user_login的要少，无法对齐。
```python
unique login users per day
 2000-01-01    11
2000-01-02    23
2000-01-03    29
2000-01-04    48
2000-01-05    62
2000-01-06    75
2000-01-07    70
2000-01-08    69
2000-01-09    49
2000-01-10    45
2000-01-11    30
2000-01-12    13
dtype: int64

 daily register users:
 2000-01-01    27
2000-01-02    24
2000-01-03    20
2000-01-04    34
2000-01-05    20
2000-01-06    30
2000-01-07    27
dtype: int64

DAU is:
 2000-01-01     38.0
2000-01-02     47.0
2000-01-03     49.0
2000-01-04     82.0
2000-01-05     82.0
2000-01-06    105.0
2000-01-07     97.0
2000-01-08      -N
2000-01-09      -N
2000-01-10      -N
2000-01-11      -N
2000-01-12      -N
dtype: float64
```
![](移动app数据分析框架\f902ffcd7cc7b493018f8aa2524780ff.png)

WAU和MAU的计算同上面的代码，只需要改变取样周期就可以了。

# Pandas用法回顾
这些pandas的方法在本文中得到使用：
* 时间相关
  - pandas.date_range: 生成一个时间序列，如pd.date_range('1/1/2000', periods = 7, freq = 'D')
  - pd.Timedelta: 两个时刻之间的差量。pd.Timedelta(1, unit = 's')，生成长度为1秒的时间差量
  - pd.DatetimeIndex: 时间序列索引，与groupeby一起使用时，可以通过它来调整索引的时间粒度，如df.groupy(pd.DatetimeIndex('column_name').date).apply(...)。将df按天进行分组。
  - Series.resample：对时间序列（索引为时刻）进行重采样。Series.resample('H')，按小时对序列进行重采样。对于求DAU, MAU, WAU等操作特别方便。
  - pd.to_datetime:将字符串表示的时间转化为datetime对象。pd.to_datetime('2000-01-01')
* 分组操作
  - DataFrame.groupby: 指定索引或者列，进行分组操作。对分组的结果可以执行apply操作。apply接受一个函数（或者lambda表达式）为参数，将分组的结果数据（也是一个DataFrame)传给这个函数。
* 构造和初始化
  - DataFrame:
```python
  df = pd.DataFrame(data = data, index = index, columns = columns)

  df = pd.DataFrame()
  df['col1'] = [...]
  df['col2'] = [...]
```
