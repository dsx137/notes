---
---

# Knuth-Morris-Pratt

>KMP

## 思想

假如有模式串为`the apple and the banana`，那么当`...and the b`匹配到`b`时匹配失败，而恰好开头有一个`the`  
既然`the`已经被匹配过契合了，就可以设计一种算法来跳过，直接把整个模式串向后移动18个字符  
让开头这个`the`对齐到原先第二个`the`的位置，避免重复匹配  

KMP怎么知道该跳多少、跳到哪呢？  不会漏掉中间的匹配吗？  
它设计了一个表，模式串的每个字符占一格  
表中的数据是模式串中 **对应字符的位置往前** ，和 **从开头往后（前缀）** 相同的最多个数  

```txt
// 模式串和文本串
the apple and this banana and the apple and the grape
the apple and this banana and the apple and this banana and the apple and the grape are delicious, then my mother told me these fruits are also healthy...

// 能这样吗？显然不能
--------------------------------------------the apple and this banana and the apple and the grape
the apple and this banana and the apple and this banana and the apple and the grape are delicious, then my mother told me these fruits are also healthy...

// 这样是完全匹配的
------------------------------the apple and this banana and the apple and the grape
the apple and this banana and the apple and this banana and the apple and the grape are delicious, then my mother told me these fruits are also healthy...
```

例子中的第四个`th`（`the grape`）确实可以对应开头的`th`（`the apple`），但它并不是最长  
如果要这么跳，那中间应该完全匹配成功的地方就被错过了  
最长的应该是`the apple and th`对应开头的`the apple and th`  

还是没看懂？对应的这个最长前缀意味着什么？它凭什么可以这么放心地往后跳这么多？  

文本串匹配失败的第一个字符记为第`i`个字符，模式串对应位置记为`j`  
将模式串向后移动然后比较`i`之前的区域，实际上也就是拿不同的前缀来比较  
将`j`之前的部分看做是模式串的子串，那就是要让子串的头和`i`之前的部分匹配  

```txt
the apple and this banana and the apple and th
the apple and this banana and the apple and th...

-------the apple and this banana and the apple
the apple and this banana and the apple and th...

-----------------the apple and this banana and
the apple and this banana and the apple and th...

------------------------------the apple and th
the apple and this banana and the apple and th...

--------------------------------------------th
the apple and this banana and the apple and th...
```

但其实`i`之前的部分已经比较过了，文本串`i`之前和模式串`j`之前的这部分是一样的  
要让子串的头匹配`i`之前的部分，那子串的头尾长得就必须一样  
反过来说，如果一个子串的头尾长得不一样，那它的头一定不匹配`i`之前的部分

搞一张表，模式串里面每一个字符占一格，表示不同子串的最后一个字符  
那个字符可能对应很多不同的前缀，即一个子串的头尾可能有多种情况相同，而且这多种情况是嵌套关系  
例如`abaababc`，其中`abaaba`就有`<a>baab[a]`、`<aba>[aba]`两种情况  
同理可以构建`1234123412341234`，就有`<1234>12341234[1234]`、`<12341234>[12341234]`、`<1234[12341234>1234]`三种情况  
对应最长的前缀时，向后移动距离最少，是最保守的，就意味着不会漏情况  
假如最长前缀之后的一个字符无法匹配，或者整个模式串匹配成功  
那这跟一开始的的情况一样了  
这个时候就把子串更新为原来的子串的最长前缀，相当于向后移动  

```txt
                                              j
the apple and this banana and the apple and the grape
the apple and this banana and the apple and this banana and the apple and the grape are delicious, then my mother told me these fruits are also healthy...
                                              i

                                              j                            
------------------------------the apple and this banana and the apple and the grape
the apple and this banana and the apple and this banana and the apple and the grape are delicious, then my mother told me these fruits are also healthy...
                                              i        

                                                                                  j
------------------------------the apple and this banana and the apple and the grape
the apple and this banana and the apple and this banana and the apple and the grape are delicious, then my mother told me these fruits are also healthy...
                                                                                  i             

                                                                                   j                                                    
-----------------------------------------------------------------------------------the apple and this banana and the apple and the grape
the apple and this banana and the apple and this banana and the apple and the grape are delicious, then my mother told me these fruits are also healthy...
                                                                                   i       

                                                                                    j                                                    
------------------------------------------------------------------------------------the apple and this banana and the apple and the grape
the apple and this banana and the apple and this banana and the apple and the grape are delicious, then my mother told me these fruits are also healthy...
                                                                                    i               
```

## 原理

### 概念

`前缀`指的是从模式串的第一个字符开始的`x`长度的字符串  
例如`abc`和`abcd`就是`abcdef`的前缀

依据对照表的思想（将操作依赖的数据记录在表里面），创建一个`next`顺序表  
用来记录模式串中每个位置对应的最长前缀长度  
（使用索引或者跳跃步数作为表的元素也可以，不过那就不叫`next`表了）  

### next 表

一个`next`表的例子可以像这样：  
`abcdxabcd`  
其中  :

| `a` | `b` | `c` | `d` | `x` | `a` | `b` | `c` | `d` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0   | 0   | 0   | 0   | 0   | 1   | 2   | 3   | 4   |

`i = 1`，`j = 0`，`next[0] = 0`

+ 循环判断`pattern[i] == pattern[j]`，`i >= len`跳出
  + 若是，则`next[i] = j + 1`，`i++`，`j++`
  + 若否，
    + 若`j > 0`，则`j = next[j]`
    + 若`j == 0`，则`next[i] = 0`，`i++`
  
表格中的值即为当前字符所对应的最长的前缀长度  
（比如第二次出现的`b`，它能对应的最长的前缀为`ab`，所以其值为`2`）

### 查找

文本串（`t`）索引为`i`，模式串（`p`）索引为`j`，两索引初始为`0`  

+ 于是进行匹配循环：
  1. 如果`t[i] == p[j]`，则`i++`、`j++`
  2. 如果模式串的第一个字符匹配失败，则`i++`
  3. 如果模式串的非第一个字符匹配失败，`j = next[j-1]`
  4. 如果模式串在文本串的当前索引一一对应，则找到字串，`j = next[j-1]`

`next[j-1]`为最后一个成功字符对应的最长前缀长度  
`j = next[j-1]`目的是将模式串索引移动到最长前缀的后一个字符，准备下次用此字符进行匹配

于是像这样，就可以在**匹配失败/匹配结束**后，直接跳到已经匹配成功的字符串（也就是最长前缀）之后

## 代码实现

这里`lps`是`最长前缀后缀（Longest Prefix Suffix）`  
不知道是哪个byd把生成`next`表的过程描述成不断截取子串了  
然后就把这个`lps`搞成`对于模式串的某个子串，最长前缀后缀是指该子串的最长前缀和后缀相同的部分`  
形似`ABCDXXXABCD`，则`lps`就为`ABCD`  
可以不用管，还是理解成最长前缀长度，只是顺便提一下

```python
def compute_lps(pattern: str) -> typing.List[int]:
    m = len(pattern)
    lps = [0] * m
    length = 0
    i = 1
    while i < m:
        if pattern[i] == pattern[length]:
            length += 1
            lps[i] = length
            i += 1
        else:
            if length != 0:
                length = lps[length - 1] ## 这里很重要
            else:
                lps[i] = 0
                i += 1
    return lps

def kmp(string: str, pattern: str) -> typing.List[int]:
    n = len(string)
    m = len(pattern)
    lps = compute_lps(pattern)
    i = 0
    j = 0
    indexes = []
    while i < n:
        if pattern[j] == string[i]:
            i += 1
            j += 1
        else:
            if j != 0:
                j = lps[j - 1]
            else:
                i += 1
        if j == m:
            indexes.append(i - j)
            j = lps[j - 1]
    return indexes
```
