---
---

# Brute-Force

## 原理

将模式串（`p`）中的字符一个一个同文本串（`t`）比较  
比较完后文本串索引向后移一个，然后重新比较，以此类推

比如可以写两个for嵌套，循环索引为`i`、`j`，初始值都设为`0`  
然后开启以下循环

1. 1. 当`j`小于模式串长度时，循环比较`t[i]==p[j]`
   2. + 若`t[i]!=p[j]`，跳出
      + 否则`j++`
2. 若`j`等于模式串长度，则找到了一个字串
3. 使`i`自加1

## 代码实现

```python
def bf(string: str, pattern: str) -> typing.List[int]:
    n = len(string)
    m = len(pattern)
    indexes = []
    i = 0
    while i <= n - m:
        j = 0
        while j < m and string[i + j] == pattern[j]:
            j += 1
        if j == m:
            indexes.append(i)
        i += 1
    return indexes
```
