# HIBLUP的学习

### 输入文件的格式

**表型文件**（phenotype file)		**.phe**

HIBLUP的表型文件除了表型记录外还包括了协变量，固定/随机效应,**文件里应该包括header **

| 第一列必须是个体ID | sex  | season | day  | weight | loc  | dam  | tr1  | tr2  | tr3  |
| ------------------ | ---- | ------ | ---- | ------ | ---- | ---- | ---- | ---- | ---- |

```
id    sex    season    day  weight  loc  	dam       tr1    	tr2    		tr3
Ind1    F    Winter    100    2.9    l3    Ind902    73.9776    21.3252    35.7737
Ind2    M    Summer    100    1.5    l2    Ind902    94.1994    19.1162    54.2453
Ind3    M    Winter    100    1.8    l4    Ind903    88.6698    5.9538    71.0284
Ind4    M    Winter    100    1.9    l1    Ind903    87.0458    -2.8179    37.2703
Ind5    M    Spring    100    2.2    l3    Ind905    68.8077    2.5357    59.5402
Ind6    F    Autumn    100    1.9    l1    Ind905    85.9242    -3.06    48.4822
```

以下所有符号在文件中均视为丢失值

```
NA  Na  .  -  NaN  NAN  nan  na  N/A  n/a  <NA>
```

应将文件分配给标记	 **--pheno .**

默认情况下，HIBLUP 将采用第二列的 trait 进行分析，用户可以通过使用—— pheno-pos n，例如—— pheno-pos 8表示，使用第8列进行分析来指定不同的列。

如果有多个性状可用于估计遗传相关性，请使用空格作为分隔符，例如：--pheno-pos 8 9 10.

对于一个性状的协变量、固定效应和环境随机效应，请分别在表型文件中指定它的位置来标记—— qcovar、—— dcovar、—— rand。

For single trait, please use comma as separator, for example:

--qcovar 4,5	: take “*day*” and “*weight*” as covariates

--dcovar 2,3	: take “*sex*” and “*season*” as fixed effects

--rand 6	: take “*loc*” as random effect

对于多个性状，如果有一些环境因素需要作为模型效应进行拟合，那么所有性状都应该在模型效应中指定，并且请在一个性状中使用<u>逗号作为分隔符</u>，在性状之间使用空格，如果没有考虑到这些因素，则在一个性状中使用0，以固定效应为例:

```bash
./hiblup ... --dcovar 2,3 0 2,3,6 ...
```

第一个性状具有两个固定效应: “性别”和“季节”; 第二个性状没有固定效应; 第三个性状具有三个固定效应: “性别”、“季节”和“地点”。



**系谱文件**（Pedigree file)	**.ped**（注意这不是plink的  .ped文件）

利用谱系文件导出分子关系矩阵和近交系数。文件中需要按顺序排列三列。第一列是个人 id，第二列和第三列分别是其父亲和母亲 id。与表型文件一样，上述列表中的任何符号出现在谱系文件中都将被视为缺失。家谱的示例文件如下:

| 个体ID | 父亲ID | 母亲ID |
| ------ | ------ | ------ |

```
id    father    mother
Ind2    NA    NA
Ind5    Ind1    NA
Ind11    NA    Ind2
Ind17    Ind11    Ind5
Ind22    Ind1    Ind5
Ind45    Ind22    Ind2
```

The file should be assigned to flag	**-- pedigree .**

第一栏中的个体 ID 的顺序可以按出生日期排序，但这不是强制性要求，因为 HIBLUP 具有对输入谱系进行排序和重新排列的自动功能。



**基因型文件**（Genotype file）

HIBLUP接受PLINK的二进制文件（.fam	.bim	.bed） ，用户可以通过 PLINK2和 TASSEL 将任何其他基因型格式(例如 VCF、 HapMap、 PED/MAP)转换为二进制格式:

```bash
# HapMap to VCF by TASSEL
./run_pipeline.pl -SortGenotypeFilePlugin -inputFile demo.hmp.txt
                  -outputFile demo.sort.hmp.txt -fileType Hapmap
./run_pipeline.pl -fork1 -h demo.sort.hmp.txt -export-exportType VCF -runfork1
# VCF to Binary
./plink2 --vcf demo.vcf --make-bed --out demo
# PED/MAP to Binary
./plink2 --ped demo.ped --map demo.map --make-bed --out demo
```

这些二进制文件应该分配给标志	**--bfile [ prefix ] .**

(1) HIBLUP 只支持加载一个二进制文件，不支持加载多个二进制文件(如全基因组按染色体单独存储) ，请在使用 HIBLUP 之前通过 PLINK 进行合并，这里有一个粗略的[指导](https://www.biostars.org/p/148657/)。

(2) HIBLUP 将 A1A1基因型编码为2，A1A2编码为1，A2A2编码为0，其中 A1是 * .bim	中每个标记的第一个等位基因。因此，估计的效应大小是在 A1等位基因上，当一个过程涉及到标记效应时，用户应该注意它

HIBLUP 没有填充功能，二进制文件中的任何缺失基因型都将被视为杂合子，这意味着在分析中缺失将被迫编码为1，如果基因型中存在缺失，我们建议用户在使用 HIBLUP 之前通过其他软件进行填充



**关系矩阵文件**（Relationship matrix file）

HIBLUP 目前只接受二进制格式的关系矩阵(XRM) ，这种格式具有最高的文件写入和数据加载效率，并且使用最少的磁盘空间，例如 demo.GA.id 和	demo.GA.bin 。

文件 	* *.id*	有一列列出所有个体号, 文件	**.bin*	存储密集矩阵的 XRM 的下三角形元素，文件	*.sparse.bin*	为稀疏矩阵存储 XRM 的行索引、列索引和三角形元素。

[默认情况下](https://www.hiblup.com/tutorials#construct-relationship-matrix-xrm)，这些文件是由 HIBLUP 中的 **--make-xrm** 生成的

此外，我们还开发了一个二进制关系矩阵的文件[格式转换器](http://hiblup.com/tutorials#format-conversion-of-xrm)，请参阅相应的章节。要通过 HIBLUP 使用二进制 XRM，只需将其前缀分配给选项  **--xrm [ prefix ] **

```bash
./hiblup ... --xrm demo.GA ...
```

如果有多个 XRM，请使用逗号作为前缀之间的分隔符:

```bash
./hiblup ... --xrm demo.GA,demo.GD ...
```

HIBLUP 具有计算 XRM 逆的功能，这可能对其他软件(如 BLUPF90，DMU，ASReml，...)有用，但对 HIBLUP 是无用的，因为 HIBLUP 使用 XRM 直接估计方差分量或解混合模型方程，因此请不要将关系矩阵的逆赋给 HIBLUP，除非你知道自己在做什么。



**基因组估计育种值文件**（Genomic estimated breeding value file）

基因组估计育种值(GEBV)用于计算加性和显性 SNP 效应，第一列为样本 id，其余列为加性或显性 GEBV，一个有三个个体的例子如下:

```
id    add
Ind1  -1.1345
Ind2  0.3245
Ind3  0.0234
```

应该将该文件分配给标记	**--gebv**

如果指定了 -- add，则应该提供加性育种值，标志 --dom 也应该提供加性育种值。如果同时指定了 --add 和 --dom，则应在文件中提供加性和显性的育种值，加性育种值的列应在显性育种值的前面



**SNP加权文件**（SNP weighting file） 	**.txt**

SNPs 加权文件用于构造加权 XRM 或计算 SNP 效应，第一列为 SNP id，其余列为 SNPs 的加性权重或显性权重，请确保所有权值均为正值，具有三个 SNPs 的示例如下:

```
SNPid  weight
SNP1   42
SNP2   0.3
SNP3   100
```

应该将文件分配给标记 **-- snp-weight**



**SNP效应文件**（SNP effect file） 	**.snpeff**

SNP效应文件用于基因组配对，至少需要5列，一个例子如下:

```
SNPid  a1  a2  freq     add       dom
SNP1   A   C   0.1243  0.30134   0.00000
SNP2   G   T   0.0345  -0.06324  0.00124
SNP3   A   G   0.3635  0.15425   -0.00913
```

其中第一列为SNP位点的名称；第二列和第三列为该SNP位点上的两个等位基因a1、a2的形式；第四列为等位基因a1的基因频率；第五列为估计的标记加性效应；第六列为估计的标记显性效应。

这个文件应该被分配到标记	**--score**

如上所述，所提供的 SNP 效应类型应该与 --add 和 --dom 指定的一致，加性 SNP 效应列应位于显性 SNP 效应列的前面。



**摘要数据文件**（Summary data file）

来自 gWAS/meta 分析的摘要数据应以 [COJO](https://yanglab.westlake.edu.cn/software/gcta/#COJO) 格式编制，见下文:

| SNP  | 效应等位基因 | 其他等位基因 | 等位基因频率 | 效应大小 | 标准误差 | P值  | 样本量 |
| ---- | ------------ | ------------ | ------------ | -------- | -------- | ---- | ------ |



```
SNP A1 A2 FREQ BETA SE P NMISS
M1 G T 0.5181 -1.565 1.155 0.1762 500
M2 A G 0.145 -1.77 1.519 0.2445 500
M3 G A 0.3206 1.498 1.583 0.3445 500
M4 C G 0.5356 0.3366 1.003 0.7374 500
M5 C G 0.0975 1.27 1.755 0.4695 500
```

列为 SNP，效应等位基因，其他等位基因，效应等位基因的频率，效应大小，标准误差，p 值和样本量。注意“ A1”需要是效应等位基因，“ A2”是另一个等位基因，“ FREQ”应该是“ A1”的频率。

应该将该文件分配给标记 **--sumstat**



**样本与SNP过滤文件**（Sample and SNP filter file）	**id.filter.txt	snp.filter.txt**

HIBLUP 可以过滤所有可用函数中的样本和 SNP，文件编写简单，只需在文件中列出个人名称或 SNP 的 ID，不需要Header。

示例文件 id.txt

```
Ind16
Ind56
Ind110
```

如果用户需要在分析中删除这些个体，只需要将其指定为标记 --remove 。相反，如果它被分配到标记 --keep，只有那些个体在文件中将用于分析。

示例文件 snp.txt

```
SNP23
SNP198
SNP432
```

同上

#### 跑完单性状模型后的输出文件

所有随机效应的估计方差和遗传力储存在tr1_pblup.vars文件中，格式如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/agHxlUibke4KpcacXpNqic2JFdY2uyarcyhibvDo4ZfDwEiabFelR93Hwqma9h3XfAYhd2zGwcdLB00zBTywQXf9yA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其中第一列为各随机效应对应的名称，包括环境随机效应，遗传随机效应和残差效应；第二列为估计的随机效应方差；第三列为随机效应方差的标准误；第四列为随机效应的遗传力；第五列为随机效应遗传力的标准误。



所有固定效应及协变量的回归系数估计值保存在tr1_pblup.beta文件中，格式如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/agHxlUibke4KpcacXpNqic2JFdY2uyarcyicHAxWj3wibFibic3wysB0ORyz3DicoiaocZlsibA140qfR6ibCej5QwhOZ7JQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

文件的第一列为均值、固定效应的各个水平以及协变量的名称；第二列为对应的估计值；第三列为估计值的标准误。



方差分析表保存在tr1_pblup.anova文件中，格式如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/agHxlUibke4KpcacXpNqic2JFdY2uyarcycoK2xgriaA9kIuzgtpx4d3PBndWibubBMYzOSCfvIickiavr4A4Kyebghg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

方差分析表的第一列为固定效应和协变量名称；第二列为自由度；第三列为总平方和；第四列为均方；第五列为F检验的统计量F值；第六列为显著性检验得到的P值，可以通过P值判断该效应的影响是否显著。



所有个体的随机效应预测值保存在tr1_pblup.rand文件中，格式如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/agHxlUibke4KpcacXpNqic2JFdY2uyarcyRK7UDX9gJwUqbUkwejghmKy8Qw1sVqsvKDd41eFLF2MEEEVicUuibW7w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其中，首列为个体ID，其后的列包括环境随机效应、遗传随机效应和残差效应的估计值。



## 异常情况总结

### **1. 结果不收敛**

遇到这种情况首先看自己跑的是 **单性状** 还是 **多性状** ，如果是单性状，按照以下思路分析：

> 单性状方差组分估计结果不收敛：
>
> 1. 是否迭代次数不够， 加大迭代次数 `--ai-maxit` 或者 `--em-maxit`
> 2. 是否方差组分估计方法不合适， 更换方法：`--vc-method TEXT:{AI,EM,EMAI,HE,HI}`
> 3. 是否数据有问题，表型数据异常值没去掉？基因组测序有问题？

如果是多性状，按照以下思路分析：

> 多性状方差组分估计结果不收敛：
>
> 1. 是否两个表型之间差别较大，比如ADG一般都是几百上千，但是FCR只有零点几。这种情况会导致不收敛，如果想要使其收敛，那么需要：**将较小的表型乘以一个常数C，使其达到与较大表型类似的值，最后再从结果中恢复（我还不知道如何恢复）**
