---
title: "bcftools学习笔记"
date: 2021-08-24T17:22:05+09:00
draft: false
cover: "/img/bcftools-cover.jpg"
images: 
  - "/img/bcftools-cover.jpg"
categories:
  - 学习笔记💻
tags:
  - Bioinfo
---

---

<!--more-->

转载并翻译国外的这篇博客：

[bcftools Primer | VP Nagraj](https://www.nagraj.net/notes/bcftools/#samplename)

另结合其他一些博客进行了补充，记录一下便于自己学习。参考链接在文末。

---

bcftools用来处理variant call（`.vcf`）格式数据。官方[操作手册](https://samtools.github.io/bcftools/bcftools.html)。

## 数据下载和预处理

1. 首先，现在1000 genome项目中20130502版本的所有文件，这些文件都是压缩文件格式（`.vcf.gz`），每个都有`.tbi`格式的索引文件
2. 下载[ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/)网站的`.vcf.gz`和`.tbi`文件
3. 创建`.vcf.gz`文件，该文件是筛选后的染色体1-22，使其仅包含ClinVar中的位点
4. 为新创建的`.vcf.gz`文件建立tabix索引

每个步骤可能需要很长时间，并且需要足够的储存容量（约20G），并且需要已经安装wget，parallel，bcftools，tabix。

```bash
## download 1000 genomes vcf files
wget ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/*.vcf.gz*

## download clinvar vcf
wget ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/vcf_GRCh37/clinvar.vcf.gz*

## use parallel to restrict each chromosome (chr1 to chr22) to clinvar sites
find . -type f -name "ALL.chr[1-9]*vcf.gz" | parallel "bcftools view {} -R clinvar.vcf.gz --output-type z --output {}.clinvar.vcf.gz"

## make sure all vcf.gz files are tabix indexed
find . -type f -name "ALL.chr[1-9]*.clinvar.vcf.gz" | parallel "tabix {}"
```

---

## 合并多个文件

如果想要连接（堆叠）多个vcf文件，可以使用`bcftools concat`，只要输入的文件共享相同的field（来源和调用都一致的应该就可以）。

现在将染色体1-22合并到一个文件中。

参数`--output-type z`表示指定的输出将被压缩：

```bash
bcftools concat ALL.chr*.clinvar.vcf.gz --output-type z --output all.clinvar.vcf.gz
```

注意：`bcftools concat`不等同于`bcftools merge`。

## view命令

`view`命令可以用于VCF和BCF格式的转换，用法如下：

```bash
bcftools view view.vcf.gz -O u -o view.bcf
```

`-O`参数指定输出文件的类型，`b`代表压缩后的BCF文件，`u`代表未经压缩的BCF文件，`z`代表压缩后的VCF文件，`v`代表未经压缩的VCF文件；`-o`参数指定输出文件的名字。

## 按名称选择单个样本

`bcftools view -s` 按照样本ID选择子集。

`all.clinvar.vcf.gz`包含多个样本。这里，将为NA20536 和 HG03718 样本创建单独的压缩文件，以及每个文件的索引（使用`bcftools index -t`）:

```bash
bcftools view -s NA20536 all.clinvar.vcf.gz --output-type z --output NA20536.clinvar.vcf.gz
bcftools view -s HG03718 all.clinvar.vcf.gz --output-type z --output HG03718.clinvar.vcf.gz

## note: bcftools index -t is equivalent to tabix here
bcftools index -t NA20536.clinvar.vcf.gz
bcftools index -t HG03718.clinvar.vcf.gz
```

## 筛选变体类型

bcftools view -v可以筛选文件，指定其包含的变体类型为"snps", "indels", "mnps", "other"。

仅包含indel类型：

```bash
bcftools view -v indels NA20536.clinvar.vcf.gz
```

```
CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	NA20536
1	978603	rs35881187	CCT	C	100	PASS	AC=2;AF=0.479233;AN=2;NS=2504;DP=14705;EAS_AF=0.8036;AMR_AF=0.6412;AFR_AF=0.0348;EUR_AF=0.5487;SAS_AF=0.5593;VT=INDEL	GT	1|1
1	984171	rs140904842	CAG	C	100	PASS	AC=2;AF=0.920527;AN=2;NS=2504;DP=7127;EAS_AF=0.9891;AMR_AF=0.9769;AFR_AF=0.7602;EUR_AF=0.9742;SAS_AF=0.9714;VT=INDEL	GT	1|1
1	1168239	rs533071750	C	CG	100	PASS	AC=0;AF=0.000599042;AN=2;NS=2504;DP=9648;EAS_AF=0;AMR_AF=0.0029;AFR_AF=0;EUR_AF=0.001;SAS_AF=0;AA=?|GGGGGGG|GGGGGGGG|unsure;VT=INDEL;EX_TARGET	GT	0|0
1	2343991	rs570192538	CCA	C	100	PASS	AC=0;AF=0.00459265;AN=2;NS=2504;DP=9045;EAS_AF=0;AMR_AF=0;AFR_AF=0.0174;EUR_AF=0;SAS_AF=0;VT=INDEL	GT	0|0
1	2435830	rs555614613	TTCC	T	100	PASS	AC=0;AF=0.00579073;AN=2;NS=2504;DP=15005;EAS_AF=0;AMR_AF=0.0029;AFR_AF=0.0204;EUR_AF=0;SAS_AF=0;VT=INDEL;EX_TARGET	GT	0|0
1	2492946	rs149579135	AG	A	100	PASS	AC=0;AF=0.00359425;AN=2;NS=2504;DP=17775;EAS_AF=0;AMR_AF=0.0014;AFR_AF=0.0129;EUR_AF=0;SAS_AF=0;AA=G|G|-|deletion;VT=INDEL	GT	0|0
.	.	.	.	.	.	.	.	.	.
```

## 根据变体ID筛选

假设我们有一下RSID，包含在一个`snps.list`文件里：

```
rs145413551
rs34610323
rs79548709
rs371163239
rs148716910
rs374704178
```

我们可以使用该文件筛选变体（SNP）：

```bash
bcftools view --include ID==@snps.list NA20536.clinvar.vcf.gz
```

```
CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	NA20536
17	648546	rs34610323	C	T	100	PASS	AC=0;AF=0.0159744;AN=2;NS=2504;DP=21874;EAS_AF=0;AMR_AF=0.0058;AFR_AF=0.0575;EUR_AF=0;SAS_AF=0;AA=C|||;VT=SNP;EX_TARGET	GT	0|0
2	31620566	rs145413551	G	T	100	PASS	AC=0;AF=0.000199681;AN=2;NS=2504;DP=19652;EAS_AF=0;AMR_AF=0;AFR_AF=0.0008;EUR_AF=0;SAS_AF=0;AA=G|||;VT=SNP;EX_TARGET	GT	0|0
21	45707000	rs374704178	G	A	100	PASS	AC=0;AF=0.000399361;AN=2;NS=2504;DP=11479;EAS_AF=0;AMR_AF=0;AFR_AF=0.0015;EUR_AF=0;SAS_AF=0;AA=G|||;VT=SNP;EX_TARGET	GT	0|0
5	151721	rs148716910	G	A	100	PASS	AC=0;AF=0.00279553;AN=2;NS=2504;DP=18789;EAS_AF=0;AMR_AF=0.0014;AFR_AF=0.0098;EUR_AF=0;SAS_AF=0;AA=G|||;VT=SNP;EX_TARGET	GT	0|0
8	1841816	rs79548709	C	T	100	PASS	AC=0;AF=0.00519169;AN=2;NS=2504;DP=16683;EAS_AF=0;AMR_AF=0;AFR_AF=0.0197;EUR_AF=0;SAS_AF=0;AA=C|||;VT=SNP;EX_TARGET	GT	0|0
8	3889458	rs371163239	T	A	100	PASS	AC=0;AF=0.000199681;AN=2;NS=2504;DP=15669;EAS_AF=0;AMR_AF=0.0014;AFR_AF=0;EUR_AF=0;SAS_AF=0;AA=T|||;VT=SNP;EX_TARGET	GT	0|0
```

## 根据染色体位置筛选

`--regions`可以根据染色体，以及染色体上的position筛选vcf文件中的变体。

比如说，我们可以单独取5号染色体的snp信息进行查看：

```bash
bcftools view --regions 5 NA20536.vcf.gz
```

```
CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	NA20536
5	40417	esv3603720;esv3603721	G	,	100	PASS	AC=0,0;AF=0.000199681,0.000798722;AN=2;CS=DUP_uwash;END=176437;NS=2504;SVTYPE=CNV;DP=16231;EAS_AF=0,0;AMR_AF=0,0;AFR_AF=0,0;EUR_AF=0,0.003;SAS_AF=0.001,0.001;VT=SV;EX_TARGET	GT	0|0
5	124186	esv3603731	T		100	PASS	AC=0;AF=0.000199681;AN=2;CS=DUP_gs;END=163795;NS=2504;SVTYPE=DUP;DP=19153;EAS_AF=0;AMR_AF=0;AFR_AF=0;EUR_AF=0.001;SAS_AF=0;VT=SV;EX_TARGET	GT	0|0
5	143490	rs142208662	C	T	100	PASS	AC=0;AF=0.00279553;AN=2;NS=2504;DP=19664;EAS_AF=0;AMR_AF=0.0014;AFR_AF=0.0098;EUR_AF=0;SAS_AF=0;AA=c|||;VT=SNP;EX_TARGET	GT	0|0
5	151721	rs148716910	G	A	100	PASS	AC=0;AF=0.00279553;AN=2;NS=2504;DP=18789;EAS_AF=0;AMR_AF=0.0014;AFR_AF=0.0098;EUR_AF=0;SAS_AF=0;AA=G|||;VT=SNP;EX_TARGET	GT	0|0
5	156288	rs193920840	C	T	100	PASS	AC=0;AF=0.000199681;AN=2;NS=2504;DP=17617;EAS_AF=0;AMR_AF=0;AFR_AF=0;EUR_AF=0;SAS_AF=0.001;AA=C|||;VT=SNP;EX_TARGET	GT	0|0
5	162045	rs568109142	G	A	100	PASS	AC=0;AF=0.000199681;AN=2;NS=2504;DP=15391;EAS_AF=0.001;AMR_AF=0;AFR_AF=0;EUR_AF=0;SAS_AF=0;AA=G|||;VT=SNP;EX_TARGET	GT	0|0
.	.	.	.	.	.	.	.	.
```

第一列可以看到全部都是5号染色体。

也可以指定更具体地区域位置，比如说10号染色体的80000:900000：

```
bcftools view --regions 10:800000-900000 NA20536.clinvar.vcf.gz
```

```
CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	NA20536
10	859076	rs144565605	T	C	100	PASS	AC=0;AF=0.000199681;AN=2;NS=2504;DP=15608;EAS_AF=0;AMR_AF=0;AFR_AF=0.0008;EUR_AF=0;SAS_AF=0;AA=T|||;VT=SNP;EX_TARGET	GT	0|0
10	860990	rs144883024	G	A	100	PASS	AC=0;AF=0.00259585;AN=2;NS=2504;DP=18990;EAS_AF=0;AMR_AF=0.0014;AFR_AF=0.0091;EUR_AF=0;SAS_AF=0;AA=G|||;VT=SNP;EX_TARGET	GT	0|0
10	871816	rs79707128	T	A	100	PASS	AC=0;AF=0.0211661;AN=2;NS=2504;DP=21039;EAS_AF=0;AMR_AF=0.0058;AFR_AF=0.0703;EUR_AF=0;SAS_AF=0.0092;AA=T|||;VT=SNP;EX_TARGET	GT	0|0
```

## 合并多个样本个体的vcf文件

利用`bcftools merge`合并多个individual样本的vcf文件：

```
bcftools merge NA20536.clinvar.vcf.gz HG03718.clinvar.vcf.gz --output-type z --output NA20536.HG03718.clinvar.vcf.gz
```

```
CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	NA20536	HG03718
1	865628	rs41285790	G	A	100	PASS	NS=2504;AA=g|||;VT=SNP;EX_TARGET;DP=33950;AF=0.00279553;EAS_AF=0;AMR_AF=0.0072;AFR_AF=0;EUR_AF=0.005;SAS_AF=0.0041;AN=4;AC=0	GT	0|0	0|0
1	879481	rs113383096	G	C	100	PASS	NS=2504;AA=g|||;VT=SNP;EX_TARGET;DP=27530;AF=0.0197684;EAS_AF=0;AMR_AF=0.0058;AFR_AF=0.0719;EUR_AF=0;SAS_AF=0;AN=4;AC=0	GT	0|0	0|0
1	880944	rs112433394	G	A	100	PASS	NS=2504;AA=g|||;VT=SNP;EX_TARGET;DP=41446;AF=0.00259585;EAS_AF=0;AMR_AF=0;AFR_AF=0.0098;EUR_AF=0;SAS_AF=0;AN=4;AC=0	GT	0|0	0|0
1	887409	rs113226136	G	C	100	PASS	NS=2504;AA=g|||;VT=SNP;EX_TARGET;DP=39832;AF=0.00119808;EAS_AF=0;AMR_AF=0;AFR_AF=0.0045;EUR_AF=0;SAS_AF=0;AN=4;AC=0	GT	0|0	0|0
1	887989	rs112966263	A	G	100	PASS	NS=2504;AA=G|||;VT=SNP;EX_TARGET;DP=36768;AF=0.00579073;EAS_AF=0;AMR_AF=0;AFR_AF=0.0219;EUR_AF=0;SAS_AF=0;AN=4;AC=0	GT	0|0	0|0
1	889450	rs58931985	C	A	100	PASS	NS=2504;AA=C|||;VT=SNP;EX_TARGET;DP=32298;AF=0.00159744;EAS_AF=0;AMR_AF=0;AFR_AF=0.0061;EUR_AF=0;SAS_AF=0;AN=4;AC=0	GT	0|0	0|0
.	.	.	.	.	.	.	.	.	.	.
```

# bcftools merge和concat区别

concat可以进行vcf的“纵”向合并，比如不同染色体的vcf文件，或者snp和indel进行的合并。但是样品顺序必须保持一致。

merge可以进行vcf的“横”向合并，比如单个样本的vcf文件的合并。

concat和merge的共同点是输入文件必须是bgzip压缩，且有索引。

## query 指定输出格式

`query`可以定制vcf文件的格式。通过表达式来指定输出格式，可定制化程度比`view`高。

-f参数通过表达式指定输出格式，其中变量的写法如下：

1. %CHROM
   代表VCF文件中染色体那一列，其他的列，比如`POS`, `ID`, `REF`, `ALT`, `QUAL`, `FILTER`也是类似的写法
2. \[ ]
   对于FORMAT字段的信息，必须要中括号括起来
3. %SAMPLE
   代表样本名称
4. %GT
    代表FORMAT字段中genotype的信息
5. \t
   代表制表符分隔
6. \n
   代表新的一行

比如`query`解析每个人的基因型：

```bash
bcftools query -f '[%CHROM\t%POS\t%SAMPLE\t%TGT\n]' NA20536.HG03718.clinvar.vcf.gz
```

```
CHROM	POS	SAMPLE	GT
1	865628	NA20536	G|G
1	865628	HG03718	G|G
1	879481	NA20536	G|G
1	879481	HG03718	G|G
1	880944	NA20536	G|G
1	880944	HG03718	G|G
.	.	.	.
```

## 编辑染色体名称

可以使用`bcftools annotate —rename-chrs`

这个命令需要提供有制表符分割文件，说明原名称和新名称，格式为“oldname\tnewname”

```
1\tchr1
2\tchr2
3\tchr3
4\tchr4
5\tchr5
6\tchr6
7\tchr7
8\tchr8
9\tchr9
10\tchr10
11\tchr11
12\tchr12
13\tchr13
14\tchr14
15\tchr15
16\tchr16
17\tchr17
18\tchr18
19\tchr19
20\tchr20
21\tchr21
22\tchr22
```

上面这个文件就表示染色体命名从1-22的数字变成“chr1”-“chr22”

```
bcftools annotate --rename-chrs chromosomes.txt NA20536.clinvar.vcf.gz
```

```
CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	NA20536
chr1	865628	rs41285790	G	A	100	PASS	AC=0;AF=0.00279553;AN=2;NS=2504;DP=16975;EAS_AF=0;AMR_AF=0.0072;AFR_AF=0;EUR_AF=0.005;SAS_AF=0.0041;AA=g|||;VT=SNP;EX_TARGET	GT	0|0
chr1	879481	rs113383096	G	C	100	PASS	AC=0;AF=0.0197684;AN=2;NS=2504;DP=13765;EAS_AF=0;AMR_AF=0.0058;AFR_AF=0.0719;EUR_AF=0;SAS_AF=0;AA=g|||;VT=SNP;EX_TARGET	GT	0|0
chr1	880944	rs112433394	G	A	100	PASS	AC=0;AF=0.00259585;AN=2;NS=2504;DP=20723;EAS_AF=0;AMR_AF=0;AFR_AF=0.0098;EUR_AF=0;SAS_AF=0;AA=g|||;VT=SNP;EX_TARGET	GT	0|0
chr1	887409	rs113226136	G	C	100	PASS	AC=0;AF=0.00119808;AN=2;NS=2504;DP=19916;EAS_AF=0;AMR_AF=0;AFR_AF=0.0045;EUR_AF=0;SAS_AF=0;AA=g|||;VT=SNP;EX_TARGET	GT	0|0
chr1	887989	rs112966263	A	G	100	PASS	AC=0;AF=0.00579073;AN=2;NS=2504;DP=18384;EAS_AF=0;AMR_AF=0;AFR_AF=0.0219;EUR_AF=0;SAS_AF=0;AA=G|||;VT=SNP;EX_TARGET	GT	0|0
chr1	889450	rs58931985	C	A	100	PASS	AC=0;AF=0.00159744;AN=2;NS=2504;DP=16149;EAS_AF=0;AMR_AF=0;AFR_AF=0.0061;EUR_AF=0;SAS_AF=0;AA=C|||;VT=SNP;EX_TARGET	GT	0|0
.	.	.	.	.	.	.	.	.	.
```

## 不显示表头查看

使用`-H`：

```
bcftools view -H NA20536.HG03718.clinvar.vcf.gz
```

## 仅查看表头

使用`-h`：

```
bcftools view -h clinvar.vcf.gz
```

## 参考链接

[bcftools的功能介绍_小青儿的生信小课堂-CSDN博客_bcftools](https://blog.csdn.net/qq_41211527/article/details/89480891)

[bcftools学习笔记(一)](https://cloud.tencent.com/developer/article/1626585)
