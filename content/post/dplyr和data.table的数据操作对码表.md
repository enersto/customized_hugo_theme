---
title: dplyr和data.table的数据操作对码表
author: 'pauke'
date: '2018-01-29'
categories:
  - r_package
  - 汉语
tags: ["R","data.table","dplyr"]
---

## 缘起

作为R中两个被讨论最多的数据操作包，dplyr 和data.table因其各具特色设计哲学俘获一批忠实用户。二者在整体风格和机制上的明显区别和完整的功能函数体系，也让二者的拥趸都能在各自舒适区呆着。

既然都摆脱了之前的舒适区，从其他数据分析工具来到R，再从basic的方案来到第三方的包，那为何不再走出当前的舒适区，了解兼通另一方呢？正所谓最好的拥趸是知己知彼，各相活用的。

## 概述

### data.table

先来看data.table包作者对于该包的目的预设和定位：

> data.table inherits from data.frame. It offers fast and nemory efficient: file reader and writer, aggregations, updates, equi, non-equi, rolling, range and interval joins, in a short and flexible syntax, for faster development.
It is inspired by A[B] syntax in R where A is a matrix and B is a 2-column matrix. Since a data.table is a data.frame, it is compatible with R functions and packages that accept only data.frames.

上面这段的重点：

* data.table是继承自data frame的操作风格；
* data.table的主干功能包括：文件读写、聚合、更新、互联（包括相等、滚动、范围、区间等）；
* data.table同时具有data.table和 data frame两种属性

data.table整体操作结构

![这里写图片描述](http://img.blog.csdn.net/20180107112657664?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUzMTcxNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### dplyr

dplyr包作者的预设与目标：

> dplyr provides a flexible grammar of data manipulation. It’s the next iteration of plyr, focused on tools for working with data frames (hence the d in the name).
> It has three main goals:
• Identify the most important data manipulation verbs and make them easy to use from R.
• Provide blazing fast performance for in-memory data by writing key pieces in C++ (using Rcpp)
• Use the same interface to work with data no matter where it’s stored, whether in a data frame, a data table or database.

重点：

* dplyr是plyr的延伸和拓展；
* 通过五个关键动词：filter， select，mutate，arrange和summarise，一个副词 group_by和重要操作符管道符号『%>%』 来构成整体框架；
* dplyr追求运算方面的速度，同样也希望有更清晰的代码可读性

dplyr的一般使用结构：

```r
    df %>% 
    filter(var1 == 5) %>% 
    group_by(var2) %>%
    select(var2,var3,var4) %>%
    mutate(newVar = var2*var3) %>%
    summarise(varMean = 
    mean(newVar, na.rm = TRUE)) %>%
    arrange(desc(varMean))
```

## 对比

### data.table转换
与dplyr相比，data.table的操作需要操作对象具有data.table这个特定的属性，在开始之前需要需要经过相应转换。data.table有这么几类转化方式：

* data.table()是类似于data.frame直接构成data.table数据的函数；

* as.data.table()则是将已有数据框通过复制转化为可供操作的函数；

* setDT()则是直接将已有数据框转换为data.table属性的数据框函数；

一般而言，setDT()是最为常用的函数，因为相比于as.data.table()，前者能够更快的完成操作，并且能够和直接的data.table操作并行执行，关于这两个函数的对比，可参考[此文](https://stackoverflow.com/questions/41917887/when-should-i-use-setdt-instead-of-data-table-to-create-a-data-table)。


```r
#实质上，d，e两个在属性上没有区别，只是二者所花费的时间和内存存在差异，这种差异在数据量大时候尤其明显
a <- letters[c(19,20,1,3,11,15,22,5,18,6,12,15,23)]
b <- seq(1,41,pi)
ab <- data.frame(a,b)
d <- data.table(ab)
e <- setDT(ab)
#setDT还可与操作并行进行
f <- setDT(ab)[a > 20,]

```

此外，data.table还会涉及到设置key variable的情况（setkey(DT,a,b)），key主要实现两个功能，一是作为索引依据，进一步加快处理计算速度，二是作为数据框的排序依据。关于key的功效可以进一步看[这篇文章](https://stackoverflow.com/questions/20039335/what-is-the-purpose-of-setting-a-key-in-data-table)。


以下对比会用到的数据框

```r
df <- data.frame(v1=1:40,
                 v2=sample(c("a","b","c","d")),
                 v3=round(rnorm(20,mean = c(20,35),sd = c(3,7)),2),
                 v4=c(1L,2L,4L,6L))
dt <- setDT(df)
setkey(dt,v1)
```

### 子集提取 extract

#### 观测值子集提取

* 提取具体行
```r
df %>%
slice(3:15)
################
dt[3:15,] #或者
dt[3:15]
```


* 提取符合逻辑条件的数据记录

```r
df %>%
filter(v3 > 20) 
################
dt[v3 >20,]

```
* 提取符合多条逻辑条件的数据记录

```r
df %>%
filter(v2 %in% c("a","d")) 
################
dt[v2 %in% c("a","d"),]

```

* 随机选取部分数据

```r
#选取成比例的数据
df %>%
sample_frac(0.5, replace = TRUE) 
################
暂时没有发现这个功能

#选取n条数据
df %>%
sample_n(10, replace = T) 
################
dt[sample(.N,10,replace = T)]
```

* 删除重复值

* 4、随机选取部分数据

```r
#选取成比例的数据
df %>%
sample_frac(0.5, replace = TRUE) 
################
暂时没有发现这个功能，但dt和df有同等的属性，此处可将df替换为dt。

#选取n条数据
df %>%
sample_n(10, replace = T) 
################
dt[sample(.N,10,replace = T)]
```

* 5、删除重复值

```r
#删除重复观察值(所有列值都重复的值）
df %>%
distinct() 
################
unique(dt)
#删除特定变量重复值
df %>%
distinct() 
################
unique(dt,list(V1,V2))
```

#### 变量子集提取

dplyr和data.table在变量子集提取上有各自的思路，dplyr通过select()中嵌套相应的函数来获取符合要求的列，data.table则通过『with = F』来实现在data.frame中j项的操作。

```r
#一般的选取
df %>%
select(v2,v3) 

#选取特定列
##选取名为x1、x2、x3、x4、x5的列
select(df, num_range("x", 1:5)) 。
##选取在Sepal.Length和Petal.Width之间的所有列(包含Sepal.Length和Petal.Width)
select(df, Sepal.Length:Petal.Width)
##选取除Species以外的所有列
select(df, -Species)

# 对变量名称有要求的选取
##选取名称中含有字符的列
select(df，contains(".")) 
##选取名称以指定字符串结尾的列
select(df, ends_with("Length"))
##选取每一列
select(df, everything())
##选取名称符合指定表达式规则的列
select(df, matches(".t."))
##选取名称在指定名字组内的列
select(df, one_of(c("Species", "Genus"))) 
##选取名称以指定字符串为首的列
select(df, starts_with("Sepal"))

################

#选取特定列
##作为向量（vector）输出
dt[,v2]
##作为data.table输出
###简单版本，适合单独提取少数列
dt[,.(v2,v3)]
###标准版本
dt[,c('v2','v3'), with = F]

# 对变量名称有要求的选取
## data.table可以通过在『j』的位置嵌套函数实现对特定名称字段的选取
dt[,grep("2|4", colnames(dt)),with = F]

```

### 综合 summarize

![这里写图片描述](http://img.blog.csdn.net/20180128161438938?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUzMTcxNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 一般的数据综合
```
#概括计算
df %>%
summarize(avg = mean(v3)) 
################
dt[,.(avg = mean(v3))]

#概括计算每一列
df %>%
summarize_each(fun(mean)) 
################
dt[,lapply(.SD, mean)]

#data.table可以更进一步的完成更为复杂的概括计算
dt[,lapply(.SD, function(x)list(mean(x,na.rm = T),sum(x))),.SDcols= -c("v2")]
dt[,lapply(.SD,sum),by=V2,.SDcols=paste0("v",3:4)]
```

* 分组数据综合

```
#分组概括计算
df %>%
group_by(v2) %>%
summarize(avg = mean(v3)) 
################
dt[,.(avg = mean(v3)),by = v2]
##分组计算多于一组时候，data.table通过扩增 by实现
dt[,.(avg = mean(v3)),by = .(v2,v4)]

```

* 用于数据综合的函数

dplyr中有一套单独用于数据综合的函数。同时，data.table在『j』的位置嵌套函数的特性，可以使用这些函数，也可以用其他来源的函数。此处列举仅作数据综合时的考虑，就不做两个包的一一对比。

```
#向量的第一个值
dplyr::first
#向量的最后一个值
dplyr::last
#向量的第n个值
dplyr::nth
#向量中元素的个数
dplyr::n
#向量中的不同元素的个数
dplyr::n_distinct
#向量的IQR(四分位距)
IQR
#向量中的最小值
min
#向量中的最大值
max
#向量中的均值
mean
#向量中的中位数
median
#向量中的方差
var
#向量中的标准差
sd
```

### 创建/更新变量 add/update

![这里写图片描述](http://img.blog.csdn.net/20180128161514951?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUzMTcxNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

data.table在变量的创建和更新方面，有比dplyr更大的范围，前者能够对已有的变量进行更新，后者只能创建新的变量。

dplyr通过mutate及其相关的函数来实现这一功能，data.table则通过『:=』实现。

* 基本更新
```
#添加单一列
df %>%
mutate(v3p = v3 + 100) 
################
dt[,v3p:= v3 + 100]

#操作多列
df %>%
mutate(v3p = v3 + 100，v3s = cumsum(v3))
################
dt[,`:=`(v3p = v3 + 100, v3s = cumsum(v3))]
##或
dt[,c("v3p","v3s"):= list(v3 + 100,cumsum(v3))]
```

* 其他添加/更新

```
# dplyr
##添加一列新函数并删除旧函数
df %>%
transmute(v3_neu = v3*100 + v1*5)
##把除第一个值以外的所有元素提前，第一个元素为NA
dplyr::lead 
```


### 合并 join

对于表合并，dplyr有专门的函数来处理，data.table则没有专门函数处理，通过内含带合并的字段来实现。

![这里写图片描述](http://img.blog.csdn.net/20180128230634704?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUzMTcxNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* left join

![这里写图片描述](http://img.blog.csdn.net/20180128230821552?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUzMTcxNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```r
left_join(a, b, by = "x1")
################
b[a]
```
* right join

![这里写图片描述](http://img.blog.csdn.net/20180128231406724?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUzMTcxNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```r
right_join(a, b, by = "x1")
################
a[b]
```

* inner join

![这里写图片描述](http://img.blog.csdn.net/20180128231622683?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUzMTcxNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```r
inner_join(a, b, by = "x1")
################
b[a,nomatch = 0]
```

* full join

![这里写图片描述](http://img.blog.csdn.net/20180128231945621?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUzMTcxNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

data.table的full join没有直接明确的支持，关于原因，可以参考[此处](https://stackoverflow.com/questions/12773822/why-does-xy-join-of-data-tables-not-allow-a-full-outer-join-or-a-left-join)， 不过依然可以通过曲折的办法实现。

```r
full_join(a, b, by = "x1")
################
unique_keys <- unique(c(a[,x1], b[,x1]))
b[a[J(unique_keys)]]
#或者
b[a[J(unique_keys)]]
```

* semi join

寻找a中与b匹配的

![这里写图片描述](http://img.blog.csdn.net/20180128235723635?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUzMTcxNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

同样，data.table没有一级的semi join方式，只有通过次级方式实现，可参考[此处](https://stackoverflow.com/questions/18969420/perform-a-semi-join-with-data-table) 。

```r
semi_join(a, b, by = "x1")
################
w = unique(a[b,which=TRUE,allow.cartesian=TRUE])
a[!!w]
```

* anti_join

寻找a中与b不匹配的
![这里写图片描述](http://img.blog.csdn.net/20180129000307556?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUzMTcxNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```r
anti_join(a, b, by = "x1")
################
暂无简洁方式
```

