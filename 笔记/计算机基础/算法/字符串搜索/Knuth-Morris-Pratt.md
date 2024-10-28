---
---

# Knuth-Morris-Pratt

>KMP

## 原理

依据对照表的思想  创建一个`next`顺序表  
用来记录模式串中每个位置对应的最长前缀长度  
（使用索引或者跳跃步数作为表的元素也可以）

`前缀`指的是从模式串的第一个字符开始的`x`长度的字符串  
例如`abc`和`abcd`就是`abcdef`的前缀

一个前缀表的例子可以像这样：  
`abcdxabcd`  
其中  :

| `a` | `b` | `c` | `d` | `x` | `a` | `b` | `c` | `d` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0   | 0   | 0   | 0   | 0   | 1   | 2   | 3   | 4   |

表格中的值即为当前字符所能放下的最长的前缀长度  
（比如第二个`b`，它能容纳的最长的前缀为`ab`，所以其对应的值为`2`）

文本串（`t`）索引为`i`，模式串（`p`）索引为`j`，两索引初始为`0`  
于是进行匹配循环：

1. 如果`t[i]==p[j]`，则`i++`、`j++`
2. 如果模式串的第一个字符匹配失败，则`i++`
3. 如果模式串的非第一个字符匹配失败，`j=next[j-1]`
4. 如果模式串在文本串的当前索引一一对应，则找到字串，`j=next[j-1]`

`next[j-1]`为最后一个成功字符对应的最长前缀长度  
`j=next[j-1]`目的是将模式串索引移动到最长前缀的后一个字符，准备下次用此字符进行匹配

于是像这样，就可以在**匹配失败/匹配结束**后，直接跳到已经确定会出现的字符串（也就是最长前缀）之后

## 注意

在生成`next`表时有一个步骤很重要  
当上一个字符的`next`值（即最长前缀长度）不为零的时候  
如果当前字符不匹配，不能直接把当前字符的`next`值置为`0`  
因为在这种情况下，意味着虽然跟着上一个字符对应的最长的那个前缀是没指望了，但是次长的还有可能  
如果直接置为`0`，则会跳过一些情况，导致结果错误

例如：

| 字符串    | 值                   |
| --------- | -------------------- |
| `pattern` | `abaababc`             |
| `text`    | `abaababaababcxxxxxxxxxxx` |

此时`abaababc`的第三个`b`的`next`值应为`2`

为什么？  
因为如果当前字符存在一个对应的长度大于2的前缀，那么上一个字符就必定包含在其中  
而上一个字符对应的前缀不止那一个最大的，其他的前缀其实暗含在`next`表中  
同一个字符对应的前缀与前缀之前其实是嵌套关系  
例如`abaababc`，其中第四个`a`对应的前缀就有`a`、`aba`两个  
同理可以构建`1234123412341234`，其中第四个`4`对应的前缀就有`1234`、`12341234`、`123412341234`三个

## 代码实现

这里`lps`是`最长前缀后缀（Longest Prefix Suffix）`  
不知道是哪个byd把生成`next`表的过程描述成不断截取字串了  
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
