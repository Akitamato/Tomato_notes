# bcftools 

## 简介

**bcftools** 是一款多种实用工具的集合，它可以用于处理VCF文件和二进制的BCF文件。它可以接受VCF格式、压缩的VCF格式以及BCF格式，并能自动检测输入的格式类型。在有索引文件存在的条件下，BCFtools 可以应用于所有场景，在没有索引文件存在时，BCFtools只能应用于一些特定的场景。BCFtools 可以用于过滤、转换、合并、统计、注释等操作。



## 合并vcf文件

使用bcftools合并文件之前，你先要生成索引文件。

```bash
bcftools index input_file1.vcf.gz
bcftools index input_file2.vcf.gz
```

有了索引文件后，bcftools才能检索相应的文件；接下来以input_file1.vcf.gz为准合并文件；

```bash
bcftools merge input_file1.vcf.gz input_file2.vcf.gz -o output_file.vcf.gz
```

## 一些题外话

尽管不属于bcftools范畴，但是这里还是有一些用于操作vcf文件的技巧，不另外开markdown了。在这里记录：



查看vcf文件的染色体顺序

```bash
grep -v "^#" Dam.analysis.imp.vcf | cut -f 1 | uniq
```

> 1. `grep -v "^#" Dam.analysis.imp.vcf`：这个子命令使用`grep`命令来搜索文件`Dam.analysis.imp.vcf`中的内容。`-v`选项表示只输出不匹配给定模式的行，而`"^#"`则是一个正则表达式，用来匹配以`#`开头的行。因此，这个子命令的作用是输出文件中所有不以`#`开头的行。
> 2. `cut -f 1`：这个子命令使用`cut`命令来处理上一个子命令的输出。`-f 1`选项表示只保留每行的第一个字段。字段之间默认以制表符分隔，但您也可以使用`-d`选项来指定其他分隔符。因此，这个子命令的作用是只保留上一个子命令输出中每行的第一个字段。
> 3. `uniq`：这个子命令使用`uniq`命令来处理上一个子命令的输出。它会删除相邻的重复行，只保留一行。因此，这个子命令的作用是删除上一个子命令输出中相邻的重复行。

查看vcf.gz的染色体顺序

```bash
zgrep -v "^#" YX_58097.vcf.gz | cut -f 1 | uniq
```

