---
layout:     post
title:      UC Berkeley 22Fall DATAC100 数据科学的基础与结构 复习笔记
subtitle:   期末复习顺便写博
date:       2022-12-5
author:     Chengrui
header-img: img/UC Berkeley.jpg
catalog: true
tags:
    - 数据科学
    - 学习笔记
---

# UC Berkeley 22Fall DATAC100 数据科学的基础与结构 复习笔记

### 1. Pandas基础知识

##### 1.1 Indexing with loc, iloc, and []

###### loc

```python
import pandas as pd
elections = pd.read_csv('Path_to_File')
elections.loc[0:4] #返回DataFrame的前5行
```

**注意！** loc[]在切片时是左闭右闭，所以

```python
elections.loc[0:4]
```

返回的值中会包括index为4的那一行

其他常用指令：

```python
elections.head()
elections.tail()
```

loc也可以用来选择某些列元素

```python
elections.loc[0:4, 'Year':'Party']
```

将会返回DataFrame前5行中从'Year'到'Party'的列

loc按照标签选择样本：

- 行的标签在DataFrame的最左边，以加粗的文本显示
- 列标签为列名

loc可以接受的变量包括：

- list

  ```python
  elections.loc[[87, 25, 179], ["Year", "Candidate", "Result"]]#返回这三列的值
  ```

- 切片（切记是左闭右闭）

  ```python
  elections.loc[[87, 25, 179], "Popular vote":"%"]#返回从Popular vote到 % 所有列的值
  ```

- 一个单独的值

  ```python
  elections.loc[[87, 25, 179], "Popular vote"]#返回Popular vote的值
  ```

如果你需要所有的行或者所有的列，你可以使用“：”

```python
elections.loc[:, ["Year", "Candidate", "Result"]]
```

###### iloc

与loc按照标签选择不同，iloc按照数字选择样本：

- 行的数字从零开始依次向后
- 列的数字从零开始依次向后

iloc可以接受的变量包括：

- list

  ```python
  elections.iloc[[1, 2, 3], [0, 1, 2]]
  ```

- 切片（这里是**左闭右开**）

  ```python
  elections.iloc[[1, 2, 3], 0:3]#返回第2-4行，1-3列
  ```

- 一个单独的值

  ```python
  elections.iloc[[1, 2, 3], 1]#返回第2列
  ```

如果你需要所有的行或者所有的列，你可以使用“：”

```python
elections.loc[:, [1, 2, 3]]
```

###### []

[]可以接受的变量包括：

- 行数字的切片

  ```python
  elections[3:7]#返回第4-7行（左闭右开）
  ```

- 列标签的list

  ```python
  elections[["Year", "Candidate", "Result"]].tail(5)#返回这三列的最后5行
  ```

- 一个单独的列标签

  ```python
  elections["Candidate"].tail(5)#返回这一列的最后5行
  ```

##### 1.2 Series and indices

[]操作返回的数据类型不是DataFrame，而是Series

###### 三种基本数据结构

pandas中有三种基本的数据结构

- DataFrame：2D的表型数据
- Series：1D的一列数据
- Index：一系列的行/列标签

###### 三种结构之间的关系

**DataFrame**是拥有相同**Index**的一系列**Series**的集合

Index可以是非数字的，同时Index可以有一个名字，比如“Year”

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8u6ebc092j31h10tjdml.jpg)

返回DataFrame而非Series的方法：

```python
elections['Year'].head()# 返回Series
elections[['Year']].head()# 返回DataFrame
```

如何得到行/列的标签

```python
elections.index #返回行标签
elections.columns #返回列标签
```

##### 1.3 条件筛选

loc和[]支持的另外一种输入是布尔值的list

```python
elections['Party'] == 'Independent' #返回一个布尔值的list
elections[elections['Party'] == 'Independent'] #返回所有‘Party’为‘Independent’的记录
```

另外一些进行条件筛选的方法：

```python
.isin
.str.startswith
.query
.groupby.filter
```

##### 1.4 一些内置的函数

- size/shape

  ```python
  elecitons.size #返回整个数据表的“面积” 在elections中为1092
  elections。shape #返回维度，在elections中为（182，6）
  ```

- describe

  ```python
  elections.describe() #返回整个数据表的一些统计参数：min，max，count等
  ```

- sample

  ```python
  elections.sample # By default without replacement
  ```

- value_counts

  ```python
  Series.value_counts() #返回每个独特的值出现的次数
  ```

- sort_values

  ```python
  Series.sort_values #返回排序过后的Series
  ```

- uniques

  ```python
  Series.unique() # 返回所有不重复的值
  ```


##### 1.5 处理字符串数据

如果我们想找出2020年在California最流行的男性新生儿名字，我们可以：

```python
babynames.query('Sex == "M" and Year == 2020')
         .sort_values("Count", ascending = False)
```

如果我们想找出最长的男性新生儿名字呢？

```python
babynames.query('Sex == "M" and Year == 2020')
         .sort_values("Name", ascending = False)
```

这行代码返回的将会是按照字母表顺序排序的数据，不是我们需要的

我们可以：

```python
babynames.query('Sex == "M" and Year == 2020')
         .sort_values("Name", key = lambda x: x.str.len(),
                      ascending = False)
```

##### 1.6 添加，修改和移除某一行的数据

如果我们没有上边的代码，我们要怎么找出最长的男性新生儿名字呢？

我们可以先添加一列“姓名长度”的数据，再按照这一列排序

```python
#create a new series of only the lengths
babyname_lengths = babynames["Name"].str.len()

#add that series to the dataframe as a column
babynames["name_lengths"] = babyname_lengths

#sorting as usual
babynames = babynames.sort_values(by = "name_lengths", ascending=False)
```

过河拆桥的时候到了，我们用完这个临时创建的列之后要怎么删除它呢？

```python
babynames = babynames.drop("name_lengths", axis = 'columns')
```

我们可以使用`.drop()`