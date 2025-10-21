---
title: "Linux shell实用命令笔记（一）"
date: 2021-10-12T16:52:05+09:00
draft: false
cover: "/img/linuxmemo1-cover.jpg"
images: 
  - "/img/linuxmemo1-cover.jpg"
categories:
  - 学习笔记💻
tags:
  - CS
---

---

<!--more-->

## 字符串操作

**`${#string}`**
返回$string的长度

**`${string:position}`**
在$string中，从$position位置之后开始提取子串

**${string:position:length}`**
在$string中，从$position位置之后开始提取$length长度的子串

**`${string#substring}`**

从变量$string开头开始删除最短匹配$substring子串

**`${string##substring}`**

从变量$string开头开始删除最长匹配$sunstring子串

**`${string%substring}`**

从变量$string结尾开始删除最短匹配$substring子串

**`${string%%substring}`**

从变量$string结尾开始删除最长匹配$substring子串

注意：在进行#或##匹配时，$string的首字符必须是被删除子串$substring的第一个字符，不然无法匹配删除；
在进行%或者%%匹配时，$string的最后一个字符必须是被删除子串$substring的最后一个字符，不然无法进行删除操作。

`${parameter/parttern/string}`
用string来替换第一个匹配的pattern

`${parameter/#parttern/string}`
从开头匹配parameter变量中的pattern，匹配上后就用string来替换匹配的pattern

`${parameter/%pattern/string}`
从结尾开始匹配parameter变量中的pattern，匹配上后就用string替换匹配的pattern

`${parameter//pattern/string}`
用string来替换parameter变量中所有匹配的pattern

## 变量匹配合并

### paste按列拼接合并

**语法：**

`paste [-s][-d <间隔字符>][--help][--version][文件...]`

**常用参数：**

- -d<间隔字符>或--delimiters=<间隔字符>　用指定的间隔字符取代跳格字符。比如说合并后列之间用*分隔就是 -d*
- -s或--serial　串列进行而非平行处理，或者说对文件进行平铺。
- [文件…] 指定操作的文件路径

可以合并多个文件，直接`paste file1 file2 file3 ...`

可以对一个文件执行串列合并，即将一个文件中的多行按列合并为一行：`paste -s file`

`paste`命令是直接拼接两个数据，与数据顺序、序号无关。

### join匹配指定列合并

`join`是将两个文件指定位置相同的行进行连接。

**语法：**

join [OPTION]... FILE1 FILE2

**常用参数：**

- -a<1或2> 除了显示原来的输出内容之外，还显示指令文件中没有相同栏位的行。
没有参数a，表示inner join，忽略不匹配的行
-a1，表示left outer join，显示左边（FILE1）所有行
-a2，表示right outer join，显示右边（FILE2）所有行
- -a1 -a2，表示full outer join，全都保留
- -e<字符串>   若[文件1]与[文件2]中找不到指定的栏位，则在输出中填入选项中的字符串。
- -i或--ignore-case   比较栏位内容时，忽略大小写的差异。
- -o<格式>   按照指定的格式来显示结果。
- -t<字符>   使用栏位的分隔字符。
- -v<1或2>   跟-a相同，但是只显示文件中没有相同栏位的行。
- -1   连接[文件1]指定的列。
- -2   连接[文件2]指定的列。
- -check-order  检查是否输入的文件已正确排序，即使可以匹配
- --nocheck-order   不检查是否输入文件是否已正确排序
- --header         将文件第一行视为标题，直接输出而不进行匹配
- -z, --zero-terminated     line delimiter is NUL, not newline

`join`需要是排序后的数据，是按照指定的、需要进行匹配的列来排序。

排序用`sort`命令。

## sort排序

sort命令对文件的行进行排序，原则是从首字符向后，依次按照ASCII码进行比较，按照升序输出。

**语法：**

sort [OPTION]... [FILE]...

**排序参数：**

- -b, --ignore-leading-blanks  ignore leading blanks
- -d, --dictionary-order      只考虑空格和字母，按字典顺序排序
- -f, --ignore-case           fold lower case to upper case characters
- -g, --general-numeric-sort  按数值大小排列
- -i, --ignore-nonprinting    consider only printable characters
- -M, --month-sort            按月份顺序排列 (unknown) < 'JAN' < ... < 'DEC'
- -h, --human-numeric-sort   按照人类可读（各种数值单位）数值大小排序(e.g., 2K 1G)
- -n, --numeric-sort        根据数值大小排序
- -R, --random-sort          随机洗牌, but group identical keys.  See shuf(1)
- --random-source=FILE    get random bytes from FILE
- -r, --reverse               改为降序排列
- --sort=WORD             sort according to WORD:
general-numeric -g, human-numeric -h, month -M,
numeric -n, random -R, version -V
- -V, --version-sort      natural sort of (version) numbers within text

**其他参数：**

- --batch-size=NMERGE   merge at most NMERGE inputs at once
- -c, --check, --check=diagnose-first  check for sorted input; do not sort
- -C, --check=quiet, --check=silent  like -c, but do not report first bad line
- --compress-program=PROG  compress temporaries with PROG; decompress them with PROG -d
- --debug      annotate the part of the line used to sort, and warn about questionable usage to stderr
- --files0-from=F       read input from the files specified by NUL-terminated names in file F; If F is - then read names from standard input
- -k, --key=KEYDEF      指定按照第几列来排序
- -m, --merge               merge already sorted files; do not sort
- -o, --output=FILE     因为`sort`命令默认将结果输出到标准输出，但可以将结果用`>`写入新文件；但如果要写入原文件，就要用`-o`参数输出。重定向会清空原文件的。
- -s, --stable              stabilize sort by disabling last-resort comparison
- -S, --buffer-size=SIZE    use SIZE for main memory buffer
- -t, --field-separator=SEP  设定间隔符，同`paste`命令的`-d`参数
- -T, --temporary-directory=DIR  use DIR for temporaries, not $TMPDIR or /tmp; multiple options specify multiple directories
- --parallel=N          change the number of sorts run concurrently to N
- -u, --unique          在输出中去除重复行
- -z, --zero-terminated     line delimiter is NUL, not newline

## 定时执行命令

`crontab`命令。

`crontab` `-l` 查看当前任务

`crontab` `-e` 修改任务列表

## sed命令

[Linux 生产环境上，最常用的一套 " Sed " 技巧](https://cloud.tencent.com/developer/news/652760)

sed命令用来处理、编辑一个或多个文件。

**语法：**

`sed [-hnV][-e<script>][-f<script文件>][文本文件]`

**参数：**

- e或--expression 以选项中指定的script来处理输入的文本文件。
- f或--file 以选项中指定的script文件来处理输入的文本文件。
- h或--help 显示帮助。
- n或--quiet或--silent 仅显示script处理后的结果。
- V或--version 显示版本信息。

**动作说明：**

- a ：新增， a 的后面接字符串，这些字符串会在新的一行出现(目前的下一行)
- c ：取代， c 的后面接字符串，这些字串可以取代 n1,n2 之间的行
- d ：删除， 后面通常不接任何值
- i ：插入， i 的后面可以接字符串，而这些字符串会在新的一行出现(目前的上一行)
- p ：打印，将某个选择的数据打印。通常 p 会与参数 sed -n 一起运行
- s ：取代，可以直接进行取代（下面会细讲格式），通常这个 s 的动作可以搭配正规表示法

（待续）

---

## cut命令字符截取

选取信息是针对行进行的。从每一行剪切字节、字符、和字段写至标准输出。如果不指定file文件，则从标准输入读取。

cut命令默认以tab制表符作为分隔符。且只能以一个字符作为分隔符。

参数说明：

- b：以字节为单位分割提取，可以指定字节范围，比如1-3表示第1到3个字节。这些字节位置会忽略多字节字符边界，除非也同时指定了n参数
- c：以字符为单位分割。如果是英文字符就都是单字节，和b参数一样，如果是双字节的汉字，就会以汉字字符为单位分割
- d：自定义分隔符，不指定该参数则为tab制表符
- f：与-d参数一起使用，指定显示分割后的哪个field
- n：取消分割多节字符。和-b参数一起使用（-nb）。如果字符最后一个字节落在-b参数指定的范围内，则被输出，否则，将被排除。换句话说就是以字节分隔但是不拆开多字节字符。

以空格分隔时：`-d ' '`


---
