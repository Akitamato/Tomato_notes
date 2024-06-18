# R  daily learning note

## 一，基本统计分析

### 2023.4.21 描述性统计分析的R实现

当我们拿到一组数据的时候，在对其进行具体的分析之前，一般会先探究这组数据的整体表现。而要直观地看到数据的整体表现，我们需要对数据做一个基本的描述性统计量分析。

相对于传统的excel表格，R赋予了我们更高的自由度以及更强大的能力。今天简单学习了几个用于描述性统计的R函数。

```R
summary()
```

summary（）无需安装任何额外的R包，可以直接调用。summary（）函数提供了最小值、最大值、四分位数和数值型变量的均值，以及因子向量和逻辑型向量的频数统计。

```R
myvars <- c("mpg", "hp", "wt")
summary(mtcars[myvars])
```

可以使用sapply（）函数来计算所选择的任意描述性统计量，它的功能十分强大，甚至可以使用自编函数。正确地使用它可以避免使用循环结构。

> `sapply()`函数的参数如下：
> - X：填向量或者向量的表达式。
> - FUN：填某个函数，这个函数会应用于每个X。
> - options：如果指定了options，它们将被传递给FUN。
>

除了基础包，还有很多R包都含有计算描述统计量的函数。比如：**Hmisc , pastecs 和 psych** 

* Hmisc 包中的 **describe()** 函数可以返回变量和观测的数量、缺失值和唯一值的数目、平均值、
  分位数，以及五个最大的值和五个最小的值

```R
install.packages("Hmisc")
library(Hmisc)
myvars <- c("mpg", "hp", "wt")
 describe(mtcars[myvars])
```

* pastecs 包中的 **stat.desc()** 函数，同样可以用来计算种类繁多的描述性统计量。

  ```R
  stat.desc(x, basic=TRUE, desc=TRUE, norm=FALSE, p=0.95)
  ```

x是一个数据框或时间序列。若basic=TRUE（默认值），则计算其中所有值、空值、缺失值的数量，以及最小值、最大值、值域，还有总和。若desc=TRUE（同样也是默认值），则计算中位数、平均数、平均数的标准误、平均数置信度为95%的置信区间、方差、标准差以及变异系数。最后，若norm=TRUE（不是默认的），则返回正态分布统计量，包括偏度和峰度（以及它们的统计显著程度）和Shapiro-Wilk正态检验结果。

* psych 包中的 **describe()** 函数，它可以计算非缺失值的数量，平均数、标准差、中位数、截尾均值、绝对中位差、最小值、最大值、值域、偏度、峰度和平均值的标准误。

```R
install.packages("psych")
library(psych) 
describe(mtcars[myvars]) 
```



## **分组计算描述统计量**

比较多组个体的观测信息的时候，关注的焦点通常为各组的描述统计量信息，而不是样本总体的信息。

对于数据的分组，可以使用 aggregate() 函数：

```R
aggregate(mtcars[myvars], by=list(am=mtcars$am), mean)
```

不过，aggregate()函数每次只能调用一个单返回值函数（平均数，标准差）。对于需要一次返回多个统计量的情况可以使用 **by（）** 函数：

```R
by(data, INDICES, FUN)
#其中data是一个数据框或矩阵，INDICES是一个因子或因子组成的列表，定义了分组，FUN是任意函数

dstats <- function(x)sapply(x, summary)
myvars <- c("mpg", "hp", "wt")
by(mtcars[myvars], mtcars$am, dstats)
```

doBy包和psych包也提供了分组计算描述性统计量的函数：

对于 doBy 包中的 **summaryBy()** 函数的使用格式为：

```R
summaryBy(formula, data=dataframe, FUN=function)
#其中的formula接受以下的格式：
#     var1 + var2 + var3 + ... + varN ~ groupvar1 + groupvar2 + ... + groupvarN  
#在~左侧的变量是需要分析的数值型变量，而右侧的变量是类别型的分组变量。function可为任何内建或用户自编的R函数。         
install.packages("doBy")
library(doBy)
summaryBy(mpg+hp+wt~am, data=mtcars, FUN=mystats)
          
```

对于 psych 包中的 **describeBy（）**函数，它描述的是与 describe（）函数描述的是一样的，不过会按照多个分组变量分层。

```R
describeBy(mtcars[myvars], list(am=mtcars$am))
```

但是与 doBy 包的 summaryBy（）不同，它不能指定任意函数。



### 4.22 频数表和列联表

频数表和列联表都是用来描述数据分布的工具，但它们的应用场景不同

* 频数表是将数据集按照某个特定列分类时观察每个类/组中数据出现次数的表；

* 列联表则是将数据集按两个或两个以上类别变量联合分组时观察数据在每个分组中出现频数的表，所以又称交叉分类表。

  简单来说，频数表只能描述一个变量的分布情况，而列联表可以同时描述多个变量之间的关系。例如，我们可以用列联表来研究两个属性变量之间是否有联系。

  频数列联表常规模式：

  ![img](https://pic4.zhimg.com/80/v2-222ae19ebdeef04a7bbf3532a3c0ea3f_720w.webp)

频率列联表常规模式：

![img](https://pic1.zhimg.com/80/v2-2d2867c9de6874d4a4915b80447fc21c_720w.webp)

| 函数                         | 描述                                              |
| ---------------------------- | ------------------------------------------------- |
| table(var1, var2, ..., varN) | 使用 N 个类别型变量（因子）创建一个 N 维列联表    |
| xtabs(formula, data)         | 根据一个公式和一个矩阵或数据框创建一个 N 维列联表 |
| prop.table(table, margins)   | 依 margins 定义的边际列表将表中条目表示为分数形式 |
| margin.table(table, margins) | 依 margins 定义的边际列表计算表中条目的和         |
| addmargins(table, margins)   | 将概述边 margins（默认是求和结果）放入表中        |
| ftable(table)                | 创建一个紧凑的“平铺”式列联表                      |

* 一维度列联表

  直接使用table（）函数生成简单的频数统计表：

  ```R
  library(vcd)
  mytable <- with(Arthritis, table(Improved)) 
  
  #频数转化为比例值
   prop.table(mytable) 
  #百分比
   prop.table(mytable) *100
  ```

* 二维列联表

  ```R
  mytable <- table(A,B)
  #交叉分类的变量在~符号的右方，公式的左侧为一个频数向量#
  mytable <- xtabs(~ Treatment+Improved, data=Arthritis)
  #边际频数
  margin.table(mytable, 1)
  #比例，1为第一个变量（Treatment)
   prop.table(mytable, 1) 
  #若不使用下标数字，则输出各单元格所占比例
   prop.table(mytable) 
  #边际频数和
  addmargins(mytable)
   addmargins(prop.table(mytable))#比例
  #选定某一变量的边际和（行）
  addmargins(prop.table(mytable, 1), 2)
  ```

使用gmodels包中的CrossTable()函数是创建二维列联表的第三种方法

```R
 library(gmodels)
 CrossTable(Arthritis$Treatment, Arthritis$Improved) 
#CrossTable()函数有很多选项，可以做许多事情：计算（行、列、单元格）的百分比；指定小数位数；进行卡方、Fisher和McNemar独立性检验；计算期望和（皮尔逊、标准化、调整的标准化）残差；将缺失值作为一种有效值；进行行和列标题的标注；生成SAS或SPSS风格的输出。#
```

* 多维列联表

table()和xtabs()都可以生成多维列联表。此外margin.table(); prop.table(); addmargins(）等函数也可以推广到高于二维的情况。

ftable()函数可以用一种紧凑的方式生成列联表

```R
mytable <- xtabs(~ Treatment+Sex+Improved, data=Arthritis)
ftable(prop.table(mytable, c(1, 2)))
```

**独立性检验**

在拿到数据的时候通常要对其变量进行独立性检验，方法有三种

* 卡方独立性检验

  使用 chisq.test() 函数

  ```
  mytable <- xtabs(~Treatment+Improved, data=Arthritis)
  chisq.test(mytable)
  ```

  

* Fisher精确检验

  使用fisher.test() 函数

  ```R
  mytable <- xtabs(~Treatment+Improved, data=Arthritis)
  fisher.test(mytable)
  ```

  

* Cochran-Mantel-Haenszel检验

  使用mantelhaen.test()函数，其原假设是，两个名义变量在第三个变量的每一层中都是条件独立的。

  ```R
  mytable <- xtabs(~Treatment+Improved+Sex, data=Arthritis)
  mantelhaen.test(mytable)
  ```

  **相关性度量**

如果独立性检验结果是两个变量不独立，也就是说它们两个相关，那么相关性大小是多少呢？可以用vcd包中的

assocstats()函数可以用来计算二维列联表的phi系数、列联系数和Cramer’s V系数。

```R
library(vcd)
mytable <- xtabs(~Treatment+Improved, data=Arthritis)
assocstats(mytable)
```



## 4.24 相关

R可以计算多种相关系数，包括Pearson相关系数、Spearman相关系数、Kendall相关系数、偏相关系数、多分格（polychoric）相关系数和多系列（polyserial）相关系数。

使用cor()函数可以计算Pearson、Spearman和Kendall三种相关系数，使用格式为：

cor(x, use= , method= )

| 参数   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| x      | 矩阵或数据框                                                 |
| use    | 指定缺失数据的处理方式。可选的方式为all.obs（假设不存在缺失数据——遇到缺失数据时将报<错）、everything（遇到缺失数据时，相关系数的计算结果将被设为missing）、complete.obs（行删除）以及 pairwise.complete.obs（成对删除，pairwise deletion） |
| method | 指定相关系数的类型。可选类型为pearson、spearman 或kendall    |

##### 2.偏相关

偏相关是指在控制一个或多个定量变量时，另外两个定量变量之间的相互关系。你可以使用ggm包中的pcor()函数计算偏相关系数。

pcor(u, S)
其中的u是一个数值向量，前两个数值表示要计算相关系数的变量下标，其余的数值为条件变量（即要排除影响的变量）的下标。S为变量的协方差阵

```R
library(ggm)
colnames(states)
pcor(c(1,5,2,3,6), cov(states))
```

#### 相关性的显著性检验

cor.test()函数可以对单个的Pearson、Spearman和Kendall三种相关系数进行检验

cor.test(x, y, alternative = , method = )

其中的x和y为要检验相关性的变量，alternative则用来指定进行双侧检验或单侧检验（取值为"two.side"、"less"或"greater"），而method用以指定要计算的相关类（"pearson"、"kendall"或"spearman"）。当研究的假设为总体的相关系数小于0 时， 请使用alternative="less" 。在研究的假设为总体的相关系数大于0 时， 应使用alternative="greater"。

```R
cor.test(states[,3], states[,5])
```



当要进行多个相关性显著检验时可以使用psytch 包的 corr,test（）函数

```R
library(psych)
corr.test(states, use="complete")
```



## 4.25t检验

### 1，独立样本的t检验

t.test(y ~ x, data)

y是一个数值型变量，x是一个二分变量。调用格式或为：

t.test(y1, y2)

其中的y1和y2为数值型向量（即各组的结果变量）

```R
library(MASS)
t.test(Prob ~ So, data=UScrime)
```

### 2,非独立样本的t 检验

非独立样本的t检验假定组间的差异呈正态分布，检验的调用格式为：
t.test(y1, y2, paired=TRUE)

```R
sapply(UScrime[c("U1","U2")], function(x)(c(mean=mean(x),sd=sd(x))))
with(UScrime, t.test(U1, U2, paired=TRUE))
```

### 3,多于两组的情况

用ANOVA



## 二，组间差异的非参数检验

t检验和方差分析（ANOVA）都有一些基本假设，如果数据不满足这些假设，那么这些方法可能不适用。

对于t检验，基本假设包括：

- 数据来自正态总体或近似正态总体。
- 如果进行两独立样本t检验，那么两个样本应该是独立的。
- 如果进行配对样本t检验，那么每对观测值应该是相关的。

对于方差分析（ANOVA），基本假设包括：

- 数据来自正态总体或近似正态总体。
- 各组之间的方差应该相等（称为方差齐性）。
- 观测值之间应该是独立的。

在这种情况下，我们可以考虑使用非参数方法，例如Mann-Whitney U检验或Kruskal-Wallis检验等。

### 1，两组的比较

若两组数据独立，可以使用Wilcoxon秩和检验，调用格式为：
wilcox.test(y ~ x, data)

其中的y是数值型变量，而x是一个二分变量。调用格式或为：
wilcox.test(y1, y2)
其中的y1和y2为各组的结果变量。
