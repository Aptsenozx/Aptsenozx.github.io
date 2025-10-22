---
title: "1000 Genome数据库学习笔记"
date: 2021-09-13T11:20:55+09:00
draft: false
cover: "/img/1KG.jpg"
categories:
  - 学习笔记💻
tags:
  - Bioinfo
  - 数据库
---

---

<!--more-->

# **1000 GENOME**

[1000 Genomes | A Deep Catalog of Human Genetic Variation](https://www.internationalgenome.org/)

## 一、简介

千人基因组计划（1000 Genomes，[http://www.internationalgenome.org/](http://www.internationalgenome.org/) ，简称1KG）于2008-2015年开启的对人的基因组进行测序一个项目，目的是建立人类突变和分型的共同数据库。虽然这个项目已经结束了，EMBL-EBI的数据中心仍然获得了Wellcome Trust的基金资助来维护和扩充这个数据库。

 IGSR（International Genome Sample Resource）想实现的目标：

- 保证公众能访问和使用1000 Genomes的数据
- 补充已发表的基因组其他信息
- 向1000 Genomes持续增加新的基因组数据

项目完成的三个阶段：

|阶段|目标|测序深度|样本|完成时间|
|:---:|:---:|:---:|:---:|:---:|
|1-low coverage|Assess strategy of sharing data across samples|2-4X|180个样本全基因组测序|2008年9月|
|2-trios|Assess coverage and platforms and centres|20-60X|2个母亲-父亲-孩子的家系测序|2008年10月|
|3-gene regions|Assess methods for gene-region-capture|50X|900个样本1000个基因|2009年6月|

这个项目包括5个大的人种，25个亚人种。目前，新版共有NA编号开头的1182个人，HG开头的1768个人。1KG项目与HapMap项目共享一部分样本，其中NA编号开头的样本很有可能就是来自HapMap项目的样本。

1KG计划的最后阶段是第三阶段，代表GRCh37上的2504个样本。随后在GRCh38上重新分析了第三阶段的数据。在这项工作之后，样本已被重新排列为高覆盖率，并且对其他相关样本进行了测序，样本总数达到3202个。

第三阶段的数据集被认为是1KG计划的最终数据发布版本。第一阶段是一个试点，在有限数量的受试者上测试了不同的测序和比对方法，进而使得第三阶段的方法更加标准化。另外，第三阶段包含来自更多种群的数据。其中第一阶段有59名受试者没有包含在第三阶段，因为他们的低覆盖率测序和/或外显子数据不符合第三阶段的标准。

第三阶段的release包含来自26个种群的2504个样本，可以按照大陆分为5个super population：

- 东亚 EAS
- 南亚 SAS
- 非洲 AFR
- 欧洲 EUR
- 美国 AMR

每个样本的super population信息可以在integrated_call_samples_v3.20130502.ALL.panel中找到。

它的官方网站有一个ppt讲得很清楚如何通过官网做的data portal来下载数据：

[How_to_Access_the_Data.pdf](https://www.genome.gov/Pages/Research/DER/ICHG-1000GenomesTutorial/How_to_Access_the_Data.pdf)

因为数据库是免费公开提供的，所以没有收集表型信息，[ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/working/20130606_sample_info/20130606_g1k.ped](ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/working/20130606_sample_info/20130606_g1k.ped) 提供样本的ID，种群，性别，家族关系的信息。

样本个体的种群（国家及地区）都有相对应的代码，对应说明如下：

### Populations

This file describes the population codes where assigned to samples collected for the 1000 Genomes project. These codes are used to organise the files in the data_collections' project data directories and can also be found in column 11 of many sequence index files.

There are also two tsv files, which contain the population codes and descriptions for both the sub and super populations that were used in phase 3 of the 1000 Genomes Project:

ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/phase3/20131219.populations.tsv

ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/phase3/20131219.superpopulations.tsv

### Populations codes

```
         CHB	Han Chinese             Han Chinese in Beijing, China
         JPT	Japanese                Japanese in Tokyo, Japan
         CHS	Southern Han Chinese    Han Chinese South
         CDX	Dai Chinese             Chinese Dai in Xishuangbanna, China
         KHV	Kinh Vietnamese         Kinh in Ho Chi Minh City, Vietnam
         CHD	Denver Chinese          Chinese in Denver, Colorado (pilot 3 only)
	
         CEU	CEPH                    Utah residents (CEPH) with Northern and Western European ancestry 
         TSI	Tuscan                  Toscani in Italia 
         GBR	British                 British in England and Scotland 
         FIN	Finnish                 Finnish in Finland 
         IBS	Spanish                 Iberian populations in Spain 
	
         YRI	Yoruba                  Yoruba in Ibadan, Nigeria
         LWK	Luhya                   Luhya in Webuye, Kenya
         GWD	Gambian                 Gambian in Western Division, The Gambia 
         MSL	Mende                   Mende in Sierra Leone
         ESN	Esan                    Esan in Nigeria
	
         ASW	African-American SW     African Ancestry in Southwest US  
         ACB	African-Caribbean       African Caribbean in Barbados
         MXL	Mexican-American        Mexican Ancestry in Los Angeles, California
         PUR	Puerto Rican            Puerto Rican in Puerto Rico
         CLM	Colombian               Colombian in Medellin, Colombia
         PEL	Peruvian                Peruvian in Lima, Peru

         GIH	Gujarati                Gujarati Indian in Houston, TX
         PJL	Punjabi                 Punjabi in Lahore, Pakistan
         BEB	Bengali                 Bengali in Bangladesh
         STU	Sri Lankan              Sri Lankan Tamil in the UK
         ITU	Indian                  Indian Telugu in the UK
```

## 二、数据查询与下载

[Data | 1000 Genomes](https://www.internationalgenome.org/data)

### 查询数据

- 通过浏览器直接查询：

    千人基因组计划 -- 基因组浏览器： [1000 Genomes Browser](http://www.ncbi.nlm.nih.gov/variation/tools/1000genomes/)

- 查询某个SNP的信息，只要将SNP的rs编号按下面的格式输入地址栏就可以：

```
http://www.ncbi.nlm.nih.gov/projects/SNP/snp_ref.cgi?rs=rs35761398
http://www.ncbi.nlm.nih.gov/SNP/snp_ref.cgi?rs=2501432
```

可以按照人种来查看这些数据：ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/data_collections/1000_genomes_project/data/

每个人的目录下面都有 四个数据文件夹

```
Oct 01 2014 00:00    Directory alignment
Oct 01 2014 00:00    Directory exome_alignment
Oct 01 2014 00:00    Directory high_coverage_alignment
Oct 01 2014 00:00    Directory sequence_read
```

可以直接看最新版的vcf文件，记录了这两千多人的所有变异位点信息。可以直接看到所有的位点，具体到每个人在该位点是否变异：ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/

它的基因型信息是通过MVNcall+SHAPEIT这个程序call出来的，具体原理见：[http://www.ncbi.nlm.nih.gov/pubmed/23093610](http://www.ncbi.nlm.nih.gov/pubmed/23093610)

而且网站还提供一些说明：ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/data_collections/1000_genomes_project/working/

### 下载数据

下载数据可以通过FTP站点进行下载。

- 1KG的FTP站点：

    [http://ftp.1000genomes.ebi.ac.uk/vol1/ftp/](http://ftp.1000genomes.ebi.ac.uk/vol1/ftp/)

- NCBI FTP Site :

    [ftp://ftp-trace.ncbi.nih.gov/1000genomes/ftp](ftp://ftp-trace.ncbi.nih.gov/1000genomes/ftp)

- Amazon S3 :

    s3://1000genomes

我仅使用过前两个站点，内容应该都是一样的。

❗注意：在Linux系统操作时，可以使用wget命令进行文件下载，格式是`wget [Link]`，这里要使用ftp地址而不是http地址。


## 三、表达相关数据

### Geuvadis的RNA测序项目

[http://www.geuvadis.org](http://www.geuvadis.org/)

关于测序细节的论文：[Lappalainen et al. Nature 2013](http://dx.doi.org/10.1038/nature12531)
包含465个个体（包括种群：CEU, TSI, GBR, FIN, YRI）的RNAseq数据（mRNA和miRNA）

（但是说明里是462 unrelated human lymphoblastoid cell line samples，和官网标题里的465样本有差别，还没弄清楚为什么）

```
CEU：具有北欧和西欧血统的犹他州居民 (CEPH)。
TSI：意大利的Toscani人。
GBR：英格兰和苏格兰的英国人。
FIN：芬兰的芬兰人。
YRI：尼日利亚的Ibadan的Yoruba人。
```

这个项目包含三个阶段的不同测序数据，阶段三应该是最全的：

- mRNA测序数据：[http://www.ebi.ac.uk/arrayexpress/experiments/E-GEUV-1/samples.html](http://www.ebi.ac.uk/arrayexpress/experiments/E-GEUV-1/samples.html) 

- miRNA测序数据：[http://www.ebi.ac.uk/arrayexpress/experiments/E-GEUV-2/samples.html](http://www.ebi.ac.uk/arrayexpress/experiments/E-GEUV-2/samples.html) 

- 完整的包含mRNA和miRNA测序数据：[https://www.ebi.ac.uk/arrayexpress/experiments/E-GEUV-3/](https://www.ebi.ac.uk/arrayexpress/experiments/E-GEUV-3/)  

#### Bam格式数据

项目提供已经处理过的数据（bam格式）下载：

Processed Data: [https://www.ebi.ac.uk/arrayexpress/experiments/E-GEUV-3/files/processed/](https://www.ebi.ac.uk/arrayexpress/experiments/E-GEUV-3/files/processed/)

（human hg19 reference genome (autosomes+X+Y+M) with the GEM mapper v 1.349 mapping entire reads）

#### eQTL结果数据

表达数据分析结果数据——eQTL（表达数量性状基因座）结果下载：

[https://www.ebi.ac.uk/arrayexpress/experiments/E-GEUV-3/files/analysis_results/](https://www.ebi.ac.uk/arrayexpress/experiments/E-GEUV-3/files/analysis_results/)

但是这里的数据是拆分开的，分为EUR和YRI，欧洲大陆4个种群和非洲大陆的尼日利亚种群。

数量性状包括：外显子，基因，转录率，转录的repetitive element，miRNA。文件中包含False discovery rate低于5%的所有关联association。如果有多个相同p值的最优关联变体，随机选择其中之一。

下面是官方对eQTL文件的一些说明：

eQTL file set:

- Sample set & sample size : EUR373, YRI89
- Quantitative trait: exon, gene, transcript ratio (trratio), transcribed repetitive element (repeat), miRNA (mi)
- Set of associations included in the file: All the associations below false discovery rate 5%, or best association per each gene (for exon, transcript ratio) or unit (gene, repeat, miRNA). If there are several best associating variants with the same p-value, one of them has been chosen randomly.

eQTL file format:

- The file contains variantâ€“QT association information, with the following columns:
1	SNP_ID : Variant identifier according to dbSNP137; position-based identifier for variants that are not in dbSNP (see Supplementary material pp 45)
2	ID : Null (-)
3	GENE_ID : Gene identifier according to Gencode v12, miRBase v18, repeats based on their start site
4	PROBE_ID : Quantitative trait identifier; the same as GENE_ID expect for:
Exons: GENEID_ExonStartPosition_ExonEndPosition
Transcript ratios: Transcript identifier according to Gencode v12
5	CHR_SNP : Chromosome of the variant
6	CHR_GENE : Chromosome of the quantitative trait
7	SNPpos : Position of the variant
8	TSSpos : Transcription start site of the gene/QT
9	Distance : | SNPpos â€“ TSSpos |
10	rvalue	: Spearman rank correlation rho (calculated from linear regression slope). The sign denotes the direction of the nonreference allele (i.e. rvalue<0 means that nonreference allele has lower expression)
11	pvalue : Association p-value
12	log10pvalue : -log10 of pvalue

### 其他表达数据

1. 60个CEU个个体RNAseq

    [http://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-197](http://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-197)

2. 800 HapMap个体的表达芯片

    [http://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-198](http://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-198)

    [http://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-264](http://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-264)

3. 69个YRI个体的RNAseq数据

    [http://www.ebi.ac.uk/arrayexpress/experiments/E-GEOD-19480](http://www.ebi.ac.uk/arrayexpress/experiments/E-GEOD-19480)


---