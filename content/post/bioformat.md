---
title: "生信数据格式整理"
date: 2021-08-17T16:27:28+09:00
draft: false
cover: "/img/bioformat/bioformat-cover.jpg"
images: 
  - "/img/bioformat/bioformat-cover.jpg"
categories:
  - 学习笔记💻
tags:
  - bioinfo
---

---

<!--more-->

## PLINK文件格式（bed, bim, fam）

GWAS分析时常用PLINK软件，PLINK文件主要有三类：bed, bim, fam。

bed：储存基因型信息

bim：储存每个遗传变异信息（SNP）

fam：储存样本信息

### bed文件

文件储存样本等位基因的信息，开头前三个字节都是0x6c, 0x1b, 0x01。这之后开始储存信息，每个字节转换成二进制表示后对应等位基因的形式。

假设遗传变异数为V，样本数为N。那么接下来就是N/4个字节的序列，如果无法整除就取整后加一作为字节数。编码信息为：

00：基因型是bim文件第一个等位基因的纯合子

01：基因型缺失

10：基因型是杂合子

11：基因型是bim文件第二个等位基因的纯合子

**例 ：**

.ped文件如下：

    1 1 0 0 1 0 G G 2 2 C C

    1 2 0 0 2 0 A A 0 0 A C

    1 3 1 2 1 2 0 0 1 2 A C

    2 1 0 0 1 0 A A 2 2 0 0

    2 2 0 0 2 2 A A 2 2 0 0

    2 3 1 2 1 2 A A 2 2 A A

.map文件如下：

    1 snp1 0 1

    1 snp2 0 2

    1 snp3 0 3

则生成的bed文件如下：

    0x6c 0x1b 0x01 0xdc 0x0f 0xe7 0x0f 0x6b 0x01

前三个字节’0x6c 0x1b 0x01’是固定格式。从第四个字节0xdc开始为基因型信息。

0xdc转换为二进制值为11011100。

读取顺序为：每个二进制值从后往前两个两个读，逐个记录各个snp位点的各个样本的情况。

11011100从后往前数，第一个snp对应’00’，也就是第一个等位基因的纯合子，所以第一个样本的第一个snp位点基因型是GG（下面的bim文件对应的第一行等位基因1是G）；然后样本2在这个位点对应’11’，所以基因型是AA；样本3是’01’，基因型缺失；样本4是’11’，基因型是AA；

然后看第五个字节0x0f，二进制值是00001111，则样本5对应’11’，样本6对应’11’，基因型都是AA。后面两个’00’表示缺失，因为总共6个样本，每个字节表示4个样本，最后会缺少两个样本。

后面的字节表示后续snp的信息。

### bim文件

没有题头的文本文件，每一行代表一个遗传变异（SNP），一共有6列。

各列含义为：

1. 染色体编号（1-22/X/Y/XY/MT, ’0’表示缺失）
2. 变异标识符，相当于每个遗传变异的编号，比如SNP采用’rs’开头的编号
3. 每个遗传变异在染色体上的位置，用摩尔根或厘摩表示
4. 碱基对的坐标
5. 等位基因1，通常是次要等位基因（minor allele）
6. 等位基因2，通常是主要等位基因（major allele）

上面的例子生成的bim文件如下：

    1 snp1 0 1 G A

    1 snp2 0 2 1 2

    1 snp3 0 3 A C

### fam文件

没有题头的文本文件，每一行代表一个样本，一共有6列。

各列含义为：

1. 家系编号（’FID’）
2. 家系内部编号（’IID’，不能是’0’）
3. 父系编号（’0’表示父系信息缺失）
4. 母系编号（’0’表示母系信息缺失）
5. 性别编号（’1’=男，’2’=女，’0’=未知）
6. 表型值（’1’=对照组，’2’=病例，’-9’/’0’表示表型缺失）

---

## SAM格式

Sequence Alignment/Map format。储存测序数据和参考基因组比对结果的文件，每行以table分隔，包含标头（header section）和比对部分（alignment section）

<div align=center>
<img src=/img/bioformat/0.png style="zoom:70%;"/>
</div>  

## BAM格式

Binary Alignment/Map format，SAM的二进制格式文件，通过BGZF library参考库压缩而成。

详细👉

[NGS数据格式梳理02-SAM/BAM格式最详细解读](https://mp.weixin.qq.com/s?__biz=MzUwOTg0MjczNw==&mid=2247483721&idx=1&sn=8bed62f0ea0520c25df6143f50a4fa7c&chksm=f90d4517ce7acc015e534c397d4c81295ee757438b222aa2be2d3e431a4303744a38246dfea5&scene=158#rd)

---

## VCF文件

用于储存变异数据的标准格式。明确，灵活，可扩展，允许将额外信息添加到信息字段。

是制表符分隔文本文件。

VCF是用于描述SNP，INDEL，和SV（结构变异位点）结果的文本文件。

<div align=center>
<img src=/img/bioformat/1.png style="zoom:100%;"/>
</div> 

VCF文件结构：
| #CHROM  | POS  | ID | REF | ALT | QUAL | FILTER | INFO | FORMAT | NA19909 |
| :----:  | :----:  | :----:  | :----:  | :----: | :----: | :----: | :----: | :----: | :----: |
|  |  |	 |  |  |  |  |  |  |  |

**#CHROME**: Chromosome, 表示变异位点是在哪个contig里call出来的，如果是人类全基因组的话就是chr1, chr2...

**POS**: Co-ordinate - The start coordinate of the variant. 变异位点相对于reference基因组所在的位置，如果是INDEL，就是第一个碱基所在的位置。

**ID**: Indentifier. variant ID。如果call出来的SNP存在dbSNP数据库里，就会显示相应的dbSNP里的rs编号；如果不是，则用“.”表示其为一个novel variant。

**REF**: Reference allele - The reference allele is whatever is found in the reference genome. It is not necessarily the major allele. 在这个变异位点处，参考基因组中所对应的碱基。

**ALT**: Alternative allele - The alternative allele is the allele found in the sample you are studying. 在这个变异位点处，研究对象的基因组variant中所对应的碱基。

**QUAL**: Score, quality score out of 100. Phred格式（Phred_scaled）的质量值，所call出来的变异位点的质量值。该值越高，则variant的可能性越大， 可信度越高。

计算方法：

① Q=-10*lgP，Q表示质量值；P表示这个位点发生错误的概率。

② Phred值Q = -10 * lg (1-p) ，p为variant存在的概率;

**FILTER**: Pass/fail - If it passed quality filters. 使用上一个QUAL值进行过滤是不够的。理想情况下，QUAL值应该是用所有错误模型算出来的，这个值就可以代表正确的变异位点了，但事实上是无法实现的。所以需要对原始变异位点做进一步过滤。无论用什么方法进行过滤，过滤完了后在FILTER都会有过滤记录，如果通过了就会是一个PASS；如果没有通过，就会注释一些信息；如果是“.”，则说明没有进行任何过滤。

**INFO**: Further information - Allows you to provide further information on the variants. Keys in the INFO field can be defined in header lines above the table. 包含variants的详细注释信息，内容非常多。以 “TAG=Value”形式给出,并使用”;”分隔的形式。其中很多的注释信息在VCF文件的头部注释中给出。以下是这些TAG的解释：

**AC，AF 和 AN**：AC(Allele Count) 表示该Allele的数目；AF(Allele Frequency) 表示Allele的频率； AN(Allele Number) 表示Allele的总数目。对于1个diploid sample而言：则基因型 0/1 表示sample为杂合子，Allele数为1(双倍体的sample在该位点只有1个等位基因发生了突变)，Allele的频率为0.5(双倍体的 sample在该位点只有50%的等位基因发生了突变)，总的Allele为2； 基因型 1/1 则表示sample为纯合的，Allele数为2，Allele的频率为1，总的Allele为2。

**DP：** reads覆盖度。是一些reads被过滤掉后的覆盖度。

**Dels：** Fraction of Reads Containing Spanning Deletions。进行SNP和INDEL calling的结果中，有该TAG并且值为0表示该位点为SNP，没有则为INDEL。

**FS：**使用Fisher’s精确检验来检测strand bias而得到的Fhred格式的p值。该值越小越好。一般进行filter的时候，可以设置 FS < 10～20。

**HaplotypeScore：** Consistency of the site with at most two segregating haplotypes

**InbreedingCoeff：** Inbreeding coefficient as estimated from the genotype likelihoods per-sample when compared against the Hard-Weinberg expectation

**MLEAC：** Maximum likelihood expectation (MLE) for the allele counts (not necessarily the same as the AC), for each ALT allele, in the same order as listed

**MLEAF：** Maximum likelihood expectation (MLE) for the allele frequency (not necessarily the same as the AF), for each ALT alle in the same order as listed

**MQ：** RMS Mapping Quality

**MQ0：** Total Mapping Quality Zero Reads

**MQRankSum：** Z-score From Wilcoxon rank sum test of Alt vs. Ref read mapping qualities

**QD：** Variant Confidence/Quality by Depth

**RPA：** Number of times tandem repeat unit is repeated, for each allele (including reference)

**RU：** Tandem repeat unit (bases)

**ReadPosRankSum：** Z-score from Wilcoxon rank sum test of Alt vs. Ref read position bias

**STR：** Variant is a short tandem repeat

**FORMAT**: Information about the following columns - The GT in the FORMAT column tells us to expect genotypes in the following columns. 关于下一列的信息。

GT：告诉我们期望的下一列的基因型。

AD：对应两个以逗号隔开的值，表示覆盖到REF和ALT碱基的reads数，相当于支持REF和支持ALT的测序深度。

DP：覆盖到这个位点的总reads数量，相当于这个位点的深度。大概一定质量值要求的reads数。

PL：对应3个以逗号隔开的值，这三个值分别表示该位点基因型是0/0，0/1，1/1的、没经过先验的标准化Phred-scaled的似然值（L）。这三种基因型的概率总和为1，如果转换成支持该基因型的概率（P），由于L=-10lgP，那么P=10^（-L/10），因此，当L值为0时，P=10^0=1。因此，这个值越小，支持概率就越大，也就是说是这个基因型的可能性越大。

GQ：表示最可能的基因型的质量值。表示的意义同QUAL。

**NA**\*\*\*\*\*: Individual identifier (optional) - The previous column told us to expect to see genotypes here. The genotype is in the form 0|1, where 0 indicates the reference allele and 1 indicates the alternative allele, i.e it is heterozygous. The vertical pipe | indicates that the genotype is phased, and is used to indicate which chromosome the alleles are on. If this is a slash / rather than a vertical pipe, it means we don’t know which chromosome they are on. 基因型的形式为0|1，0表示参考等位基因（REF），1表示替代等位基因（ALT），即表示杂合。|表示基因型是定相的，用于表示等位基因在哪条染色体上。如果是/则表示不知道位于哪个染色体上。NA\*\*\*\*\*是样本的名称，由BAM文件中的@RG下的SM标签决定的。

## 参考链接

[初探PLINK文件格式（bed，bim，fam）](https://zhuanlan.zhihu.com/p/128653464)

[生物基因数据文件--vcf格式详解_vickyleexy's blog-CSDN博客_vcf格式](https://blog.csdn.net/u012150360/article/details/70666213)

---