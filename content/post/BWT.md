---
title: "BWT算法总结（未完待续）"
date: 2021-07-05T23:08:30+09:00
draft: false
cover: "/img/BWT-cover.jpg"
images: 
  - "/img/BWT-cover.jpg"
categories:
  - 学习笔记💻
tags:
  - CS
---

---

<!--more-->

## Burrows-Wheeler Transform

该算法将原文本（序列）转换为相似的文本（序列），转换后相同的字符位置连续相邻。便于之后使用Move-to-front transform或游程编码（Run-length encoding）等对文本进行压缩。

BWT的pseudo code如下：

```
function BWT　(string s)
		create a table, where the rows are all possible rotations of s;
		sort rows alphabetically;
		return (last column of the table s')
```

简单来说，这个算法是输入一个序列s，经过一系列变换后，返回其所创建表格的最后一列s'。这个最后一列就是转换后序列。

| Index 1  | Rotate  | Index 2 | Sort |
| :----:  | :----:  | :----:  | :----:  |
| 0 | banana$ |	6 | $banana |
| 1	| anana$b | 5 | a$banan |
| 2	| nana$ba |	3 | ana$ban |
| 3	| ana$ban | 1 | anana$b |
| 4	| na$bana | 0 | banana$ |
| 5	| a$banan |	4 | na$bana |
| 6	| $banana | 2 | nana$ba | 

该算法是基于后缀和字典排序。以banana为例：

- 首先给所要转换的序列s末尾添加一个$，该标识在字典排序中小于任何其他字符。banana添加$后变成banana$；

    可以理解为$之前表示一个后缀，将该序列向左依次旋转，得到的就是Rotate列；

- 然后对Rotate列按照首字母进行排序，得到Sort列，表格中Index 2列表示Sort中的字符串在Rotate中对应的位置；
- 输出Sort列中的最后一列字符annb$aa。（这句话有点绕，就是按顺序输出Sort列中所有字符串的最后一个字符）

这样banana经过BWT转换后得到的结果就是annbaa。（这里带不带$应该是都可以，不影响）

## BWT Code

下面是BWT转换的python代码。

首先输入一串字符串，然后在句尾加上一个$字符：

```python
a = ''
words = []
a = input("Enter a string:")
a = a + '$'
print(a)
```

这里删除list变量是因为在Jupyter Notebook中，如果多次重复运行这个代码时，总会显示list这个变量有冲突，所以在这里先进行删除。第一次运行的时候不用删除就可以顺利运行。

```python
del list
words = list(a)
```

这一步输出旋转序列（Rotate列）和排序后的序列（Sort列）：

```python
table=[a[i:]+a[:i] for i in range(len(a))]
print("table:")
print(table)
table_sorted=table[:] #there must add [:]!
table_sorted.sort()
print("table_sorted:")
print(table_sorted)
print('\n')
```

然后依次找在排序后的序列table_sorted中的字符串对应在原Rotate序列table中的哪一项，输出相应的index。

```python
indexlist=[]
for t in table_sorted:
    index1 = table.index(t)
    if index1 < len(a)-1:
        index1 = index1+1
    else:
        index1 = 0
    index2 = table_sorted.index(table[index1])
    indexlist.append(index2)
    print(index1,index2)
r = ''.join([row[-1] for row in table_sorted])

print("r:")
print(r)
print("indexlist:")
print(indexlist)
```

BWT常用于基因测序数据的压缩。
由于基因序列（ATCG）中存在大量重复的字符串，比如CGCGCGCGATCG这段序列，CG就重复出现多次。这时使用BWT算法进行转换后，就可以将相同的字符转换到相邻位置，便于后续的压缩。压缩方法后面会进行补充。

还是以banana为例子，可以进行一下测试：

下面是输入'banana'的结果：
```
Enter a string:banana
banana$
```

```
r:
annb$aa
indexlist:
[4, 0, 5, 6, 3, 1, 2]
```
下面是输入两个'banana'的结果：
```
Enter a string:bananabanana
bananabanana$
```

```
r:
annnnbba$aaaa
indexlist:
[8, 0, 7, 9, 10, 11, 12, 5, 6, 1, 2, 3, 4]
```
下面是输入三个'banana'的结果：
```
Enter a string:bananabananabanana
bananabananabanana$
```

```
r:
annnnnnbbbaa$aaaaaa
indexlist:
[12, 0, 10, 11, 13, 14, 15, 16, 17, 18, 7, 8, 9, 1, 2, 3, 4, 5, 6]
```
可以看到随着序列的重复次数增多，转换后会有连续的'n'和'a'，这对于后续压缩储存是非常有利的。比如说利用Move to front，Run length encoding，或Huffman encoding进行压缩。

BWT转换除了允许压缩之外，还有很多用处。矩阵的每一行都是原序列的一个后缀

## BWT的还原

一个直观的理解就是，因为我们得到的最后一列s'：annb$aa，经过排序后就是Sort矩阵的第一列：$aaabnn，而又因为是向左旋转得到的，其实第一列就是最后一列的后一列（可以把矩阵旋转想象成一个立体的圆柱体 (￣ ︶ ￣）！所以把第一列加到最后一列（s'）的后面，然后对此时的第一列进行排序：

<div align=center>
<img src=/img/BWT/bwt1.png style="zoom:80%;"/>
</div>  

如第二个表中所示，排序后第一列又变成了原来的Sort矩阵第一列$aaabnn了。

重复这个过程，再次把s'：annb$aa加到当前这个矩阵前面，相当于把Sort矩阵的前两列移到了最后一列后面。依次类推，每次都将最后一列s'加到当前矩阵前面，然后排序，直到得到与原序列长度相同的序列，也就是将s'加到了前（n-1）列之前（n为原序列长度）。

<div align=center>
<img src=/img/BWT/bwt2.png />
</div>  
<div align=center>
<img src=/img/BWT/bwt3.png style="zoom:80%;"/>
</div>  

这样在最后一次排序后，第一行就是原序列。

这种方式所需要的空间是原序列长度的平方，而且不断地排序也需要花费时间成本。
利用LF mapping（Last to First mapping），可以更高效的进行还原。而且这种方式在FM Index中也很关键。FM Index是一种基于BWT的快速文本搜索算法，之后再进行详细介绍。

## BWT还原 - LF mapping

再回到这个表格，F列表示Sort矩阵的First列，L列表示Sort矩阵的Last列：

| Index 1 | Rotate | Index 2 | Sort | F | L |
| :-: | :----: | :-: | :----: | :-: | :-: |
| 0 | banana$ |	6 | $banana | $ | a |
| 1	| anana$b | 5 | a$banan | a | n |
| 2	| nana$ba |	3 | ana$ban | a | n |
| 3	| ana$ban | 1 | anana$b | a | b |
| 4	| na$bana | 0 | banana$ | b | $ |
| 5	| a$banan |	4 | na$bana | n | a |
| 6	| $banana | 2 | nana$ba | n | a |

因为是循环向左旋转，所以同一行中，L是F的下一个元素。


## BWT还原 python code （LF Mapping）
```python
def ibwt(r, indexlist):
    s=""
    x = indexlist[0]
    for _ in r:
        s = s + r[x]
        x = indexlist[x]
    s = s.strip('$')
    return s
print(ibwt(r, indexlist))
```




## 参考

<https://blog.csdn.net/fjsd155/article/details/97892480>
<https://en.wikipedia.org/wiki/FM-index>
<https://blog.csdn.net/g863402758/article/details/81385402?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-1&spm=1001.2101.3001.4242>

---