---
typora-root-url: ./D:\Typora\typro image
---

# Genotype text files

**.bed** ; **.bim** ; **.fam** 三个文件在分析时必须在同一个目录内

## .fam文件

![QQ图片20221123102943](/../../../PLink学习笔记.assets/QQ图片20221123102943.png)

每一行代表一个动物个体

| 家族id(品种)(FID) | 个体号(IID) | 父亲id | 母亲id | 性别：1雄，2雌（与父母id一样，0表丢失） | 表型值（0与-9表示丢失） |
| ----------------- | ----------- | ------ | ------ | --------------------------------------- | ----------------------- |



## .bim文件

![QQ图片20221123103143](/../../../PLink学习笔记.assets/QQ图片20221123103143.png)

每一行代表一个SNP，若第第五第六列中某一列出现了数字0以及另一个碱基则表明该基因型纯合。

| SNP所在的染色体号数（0表示丢失） | SNP （Marker）id | Centimorgans | Physical position(碱基对坐标) | Minor Allele | Major Allele |
| -------------------------------- | ---------------- | ------------ | ----------------------------- | ------------ | ------------ |

## .bed文件：为二进制文件



## .map文件

![QQ图片20221123105452](/../../../PLink学习笔记.assets/QQ图片20221123105452.png)

主要记录变异位置信息，无Header

| 染色体ID | Snp id | 厘摩位置 | 碱基对坐标 |
| -------- | ------ | -------- | ---------- |



## .ped文件

![QQ图片20221123110248](/../../../PLink学习笔记.assets/QQ图片20221123110248.png)

前六列等同于.**fam**文件，第六列之后每**两列**代表一个snp（如1A1A，1A2a, 2a2a...）



| Family ID # 如果没有, 可以用个体ID代替 | Individual ID | Paternal ID | Maternal ID | Sex (1=male; 2=female; other=unknown) | Phenotype (0=unknown; 1=unaffected; 2=affected) | 为SNP[分型数据](https://www.zhihu.com/search?q=分型数据&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A3120546175}), 可以是AT CG或11 12, 或者A T C G或1 1 2 2 |
| -------------------------------------- | ------------- | ----------- | ----------- | ------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------ |

> 若想将SNP分型数据从AT CG转换成为11 12可以用参数：
>
> ​		`--recode 12`
>
> 反之，用
>
> ​		`--recode AD`

# 1.Plink的使用

<u>在使用plink前，建议先把源码加注释写入记事本内，以便今后查看</u>

在linux中你需要使用**./plink**+一些参数命令来运行plink，每行都要。

对**plink**进行命令操作时，要保证它所需的文件（.**fam** ; .**bim** ; .**bed**）与.**exe**文件在一个文件夹内,然后再依如下方式输入命令：

![QQ图片20221126171804](/../../../PLink学习笔记.assets/QQ图片20221126171804.png)



## 1.2plink的基本命令。

**--bfile **前缀：使用这个前缀plink便可以<u>自动利用三个启动文件的所有内容</u>，但是该前缀的优先程度依旧没有三个各自分别的前缀高。具体如下图：

**--file** 参数：可以使用正常的plink文件，如	.map	和	.ped	文件。

![image-20221126181118914](/../../../PLink学习笔记.assets/image-20221126181118914.png)

**--chr-set** 前缀：用以指定染色体组信息，第一个参数指定二倍体生物**常染色体对**的数目（或单倍体染色体数目），不支持多倍体生物数据。具体操作如下图：

![image-20221126195120610](/../../../PLink学习笔记.assets/image-20221126195120610.png)

**"--chr-set n(X染色体/对数目) n2（Y...）n3(XY...,(X上的[拟常染色体区](https://zh.wikipedia.org/wiki/%E5%81%BD%E9%AB%94%E6%9F%93%E8%89%B2%E9%AB%94%E5%8D%80)) n4(mt线粒体..)"**



**--nonfounders** 前缀：绕过任何缺失父母信息的动物，拒绝将其视为祖代。可以在所有命令中加入。

![image-20221126201431527](/../../../PLink学习笔记.assets/image-20221126201431527.png)

**--allow-no-sex **前缀：允许利用性别缺失个体的表型。默认开启但是建议同样万金油，具体如下图：

![image-20221126202252245](/../../../PLink学习笔记.assets/image-20221126202252245.png)

**--recode** 前缀：输出*.ped&.map*文件（可用**--file**读取），它还可以输出许多别的软件需要的[格式文件](https://www.cog-genomics.org/plink/1.9/data#recode)

**--out** 前缀：+想要输出的文件名，便可以指定新输出文件命名。

`--keep` ：+加以家族ID（第一列），个体号（第二列）的文件，便可以该文件为界限，删除所有该文件外的样本

`--remove` ：与`--keep` 相反，具体如下图：

![image-20221208162606858](/../../../PLink学习笔记.assets/image-20221208162606858.png)

`--extract` : + 一列SNP ID文件， 便可以以该文件为界限，筛选所有文件内的SNP

`--exclude`： 与 `--extract` 相反



### 1.2.1plink的基本质控命令

`--geno`前缀：+定义参数X，便可过滤掉所有缺失率达到X的SNP（默认是*0.1*）

`--mind` 前缀：+定义参数X，便可过滤掉所有缺失率达到X的个体（默认是*0.1*）

.e.g：![image-20221203152553844](/../../../PLink学习笔记.assets/image-20221203152553844.png)

`--prune`前缀：过滤掉所有缺失表型数据的样本

`--maf` 前缀：为所有SNP设置一个MAF门槛（默认为*0.01*），低于此参数的SNP将会被过滤

`--max-maf` 前缀：强加一个MAF最大值门槛。

`--mac` 前缀：强加一个MAF最小计数

`-max-mac` 前缀：强加一个MAF最大计数

.e.g ：![image-20221203154429864](/../../../PLink学习笔记.assets/image-20221203154429864.png)

警告！ --maf 选项不在创始人动物中考虑，即那些父母不明的动物。 请注意，出于方便或不可用的原因，父母信息多次从数据集中被跳过。 要将此类动物纳入质量控制，请在脚本中使用 --nonfounders 选项。

`--hwe` 前缀：+参数X，过滤掉所有 Hardy-Weinberg 平衡精确检验 p 值低于提供的阈值（X）的变体。

`--autosome` 前缀：排除所有未定位和非常染色体变异，可以用于筛选有效的常染色体区域。

`--autosome-xy` ：在--autosome的基础上不排除X的拟常染色体区域。

`--thin x` : 可以用于对**标记数量**进行随机抽样，x表示随机抽样的比例，如要抽10%则x=0.1

`--thin-indiv x`:和上面的参数类似，不过这个参数是用以抽取**个体数量**的。

`--seed 整数`：可以设定一个随机数种子，储存你随机抽样的结果。

例如，如果你想用123456作为随机数种子，你可以用以下命令(无论运行多少次都是一样的output：

```bash
plink --vcf input.vcf --thin 0.1 --recode vcf --out output.vcf --seed 123456
#注意，单独使用--thin的话个体数量是不变的，如上这个文件，你再把它转化成为plink的二进制文件，map文件里还是100%的个体数量
```

### 1.3常用的plink操作

**统计多态性**

使用 `--freq` 参数我们可以得到个包含每种等位基因及其在给定群体中的频率的表格如：

```bash
CHR     SNP     A1      A2      MAF     NCHROBS
1       rs1234  A       G       0.3     1000
#如果MAF等于1或者0时代表该位置没有多态性，而NCHROBS列表示观察到的染色体数，即用了多少条染色体来计算这个位点。
```

```bash
plink --vcf YX_rf5k.vcf --freq --out YX_rf5k_freq
```

#### 1.3.1格式转换

**将正常的plink格式文件转化成为二进制文件**

假如我有a.map和a.ped两个文件

```bash
./plink --file -a --out b
```

我就得到了b.bim 	b.bed	b.fam	b.log

**「注意：」**

- 如果染色体超过23，比如30对染色体，需要设定`--chr-set 30`
- 如果有非数字染色体，比如性染色体，需要设定`--allow-extra-chr`
- 常用的动物都有对应的参数，直接设定相关动物就行，比如牛的`--cow`，下面是其它动植物的。如果没有对应的物种，直接设置染色体的条数以及允许非数字染色体即可。

**将plink二进制格式转换为正常的格式**

现在有了b.bed	b.bim	b.fam	b.log

```bash
./plink --bfile b --recode --out c
```

得到文件 c.map	c.ped

**将正常plink文件转化为vcf文件**

现在有c.map	c.ped

```bash
./plink --file c --recode vcf --out d
```

**二进制plink文件转化为vcf文件**

有b.bed	b.bim	b.fam	b.log

```bash
./plink --bfile b --recode vcf --out e
```

​																							vcf文件格式

![img](/../../../PLink学习笔记.assets/image-20220707163759450.png)

**vcf文件转化为plink文件**

现有文件 e.vcf

```bash
plink --vcf e.vcf --recode --out f
##这是转化成为正常的plink文件：f.map	f.ped
```

```bash
plink --vcf e.vcf --out g
##这是转化为二进制的plink文件：g.bed	g.bim	g.fam	
```

#### 1.3.2 VCF格式与PLINK

**使用标记ID过滤筛选vcf文件**

使用`--extract`参数可以很方便地在vcf文件中筛选出想要的SNPid

```bash
plink --file YX_rf_all_de_missing --extract SNPeff_id.txt --recode vcf --out YX_rf_all_de_missing_snpeff
#需要提取准备好用于筛选的.txt文件，格式为一列SNPID，没有表头
```

> `--keep`参数和`--extract`参数都是plink中用于筛选数据的参数，但它们的作用不同。`--keep`参数用于保留指定的样本，而`--extract`参数用于保留指定的SNP。例如，如果您想要保留vcf文件中的某些样本，则可以使用`--keep`参数。如果您想要保留vcf文件中的某些SNP，则可以使用`--extract`参数。

#### 1.4 等位基因翻转的处理办法

在操作VCF和plink文件的时候，要注意一个现象，**等位基因翻转(Allele flip）**：造成这种现象的原因是因为plink（1.9版本）默认将VCF文件中的REF列 设置为 Minor allele(A1)。

这会有什么影响吗？不会，因为在VCF文件中 “0/1”并不表示一个具体的序列，不管是REF = A ALT = G 还是 REF=G ALT = A 都是一样的。

尽管PLINK自己可以识别这种颠倒，**但是一些软件需要比对allele一致，比如HIBLUP的选配模块**

因此我们想保持VCF文件里的等位基因位置的话，需要使用参数

`--keep-allele-order` #这个最好搭配`--vcf` 一起使用

如果你的文件不是VCF文件，那么你可以使用这个参数

`--a2-allele <filename> [A2 allele col. number] [variant ID col.] [skip]` 来将 Major allele(A2）设置为与REF一致

即`--a2-allele <uncompressed VCF filename> 4 3 '#'`

如果你想将（A1）Minor allele设置为与ALT 一致可以使用

`--a1-allele <filename> [A1 allele col. number] [variant ID col.] [skip]`

需要注意的是，`--a1-allele` 参数只能用于指定非参考等位基因，而不能用于指定参考等位基因。如果你想指定参考等位基因，应该使用 `--a2-allele` 参数。



如果你用了这些参数发现还是由翻转的snp的话可以使用`--flip`参数来解决。

```bash
plink --bfile Boar_Dam_A2order --flip flip.txt --keep-allele-order  --make-bed --out Boar_Dam_A2order

```

这里的 `flip.txt`是用R写出来的，具体脚本如下

```R
rm(list = ls())
getwd()
setwd("d:/warehouse")
library(tidyverse)

prents  <- read.table("Boar_Dam_A2order.bim", col.names =c("chr", "snp", "cm", "bp", "a1", "a2") )
DLX  <- read.table("DLX_QC.bim", col.names = c("chr", "snp", "cm", "bp", "a1", "a2"))

#筛选两个bim文件中共同的marker
merged_data <- merge(prents, DLX, by = "snp") 

#筛选出需要翻转的SNP
flip_snps <- merged_data[merged_data$a1.x != merged_data$a1.y  | merged_data$a2.x != merged_data$a2.y, ] %>%
  as.data.frame()

#翻转SNP使其与标记效应一致
prents[prents$snp %in% flip_snps$snp, c("a1", "a2")] <- prents[prents$snp %in% flip_snps$snp, c("a2", "a1")] 
#写出翻转后的文件，注意替换源文件再跑选配。
  write.table(prents, file = "Boar_Dam_A2order_R.bim", col.names = F, row.names = F, quote = F) 

###这里展示了一个完全用R进行翻转的脚本，因为用--flip参数, 会出现将A/G被Plink变成了C/T的情况，具体原因不明。
```



### 2.[**plink合并文件并更新SNP位置**](https://www.cnblogs.com/chenwenyan/p/10276217.html)

一，合并文件

plink的合并需要用“merge"参数

如果是合并 	.ped	和	.map	格式文件：

```bash
plink --file data1 --merge data2.ped data2.map --recode --out merge
```

合并二进制文件和	.ped	和	.map	格式文件：

```bash
plink --bfile data1 --merge data2.ped data2.map --make-bed --out merge
```

合并文件都是二进制

```bash
plink --bfile data1 --bmerge data2.bed data2.bim data2.fam --make-bed --out merge
```

合并多个文件：

```bash
/plink-1.07-x86_64/plink --noweb --bfile file --merge-list batch.txt --make-bed --out batch
```

​	batch.txt的文件格式：

```
file1.bed file1.bim file1.fam

file2.bed file2.bim file2.fam
```

二，更新SNP位置

假设更新 rs10002到位置580000，如下所示：

原始文件：

```
	...
     rs10001   500000
     rs10002   580000
     rs10003   540000
     rs10004   560000
     ...
```

新的文件：

```
	 ...
     rs10001   500000
     rs10003   540000
     rs10004   560000
     rs10002   580000
     ...
```

更新SNP位置可以采用plink的“`--update-name` ”和“`--update-chr`”参数

具体命令：

```
./plink --bfile mydata --update-map rsID.lst --update-name --make-bed --out mydata2
```

or

```
./plink --bfile mydata --update-map chr-codes.txt --update-chr --make-bed --out mydata2
```

rsID.lst的输入格式如下：

```
    SNP_A-1919191   rs123456
    SNP_A-64646464  rs222222
    ...
```

chr-codes.txt的输入格式如下：

```
   rs123456     1
   rs987654     18
   rs678678     X
   ..
```



```R
#使用PLINK计算pca并用R实现可视化#

#清理工作区间变量#
rm(list=ls())
#设置工作区间#
setwd("d:/Rdata")
library(tidyverse)

#读取目标.fam文件#
goats <- read_tsv("ADAPTmap_genotypeTOP_20160222_full.fam", col_names = F)

#筛选需要进行pca分析的品种，%>%为管道函数，将前一步的结果直接赋值给下一步省去多步赋值减少内存#
selectedAnimals <- goats %>%
  filter(X1=="TOG" | X1=="RAN" | X1=="TED") %>%
  write_delim("selectedAnimals.txt",col_names = F)  

#进行质量控制,str_c()函数将字符组合在一起方便阅读，注意每一行的逗号前要加空格否则plink会报错#
###这里使用--autosome是为了去除性染色体的干扰，--keep是以该文件设置一个范围，删除该文件外所有的样本###
system(str_c("./plink --bfile ADAPTmap_genotypeTOP_20160222_full --chr-set 29 --autosome ",
             "--keep selectedAnimals.txt --nonfounders ",
             "--geno 0.1 --mind 0.1 --maf 0.05 ",
             "--make-bed --out afterQC")) 
       
#使用PLINK进行pca计算，--pca，在plink中使用这个参数会输出特征值与特征向量#
system("./plink --bfile afterQC --chr-set 29 --pca --out plinkPCA")

###
#可视化pca结果
###

#读取PLINK输出的结果文件#
eigenValues <- read_delim("plinkPCA.eigenval", delim = " ", col_names = F)
eigenVectors <- read_delim("plinkPCA.eigenvec", delim = " ", col_names = F)

#计算每个特征向量的方差贡献度#
eigen_percent <- round((eigenValues/(sum(eigenValues))*100),2)



# PCA 画图
ggplot(data = eigenVectors) +
  geom_point(mapping = aes(x = X3, y = X4, color = X1, shape = X1), size = 3, show.legend = TRUE ) +
  geom_hline(yintercept = 0, linetype="dotted") +
  geom_vline(xintercept = 0, linetype="dotted") +
  labs(title = "筛选后的山羊PCA图",
       x = paste0("PC1 (",eigen_percent[1,1]," %)"),
       y = paste0("PC2 (",eigen_percent[2,1]," %)"),
       colour = "Goat breeds", shape = "Goat breeds") + #这一步是在给show.legend的输出结果命名
  theme_minimal()

```

最终结果：

![image-20221208162942150](/../../../PLink学习笔记.assets/image-20221208162942150-1677845030059-3.png)











































