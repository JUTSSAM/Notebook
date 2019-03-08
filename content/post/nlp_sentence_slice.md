---
title: "[NLP]中文分词技术"
date: 2019-03-08T16:04:43+08:00
draft: true
---

[TOC]

中文分词技术主要分为三中流派：

规则分词，统计分词，混合分词（规则+统计）。

其中规则分词通过人工维护一个词库，对句子中内容进行匹配。统计分词是将统计机器学习算法应用与分词当中。实践中多用两者结合，即混合分词。

## 规则分词

基于规则的分词是一种机械分词方法，通过维护词典，在切分语句时逐个进行匹配，找到则切分，否则不予切分。

按照匹配切分的方式，分为三种：正向最大匹配法，逆向最大匹配法，双向最大匹配法。

### 正向最大匹配法

Maximun Match Method，MM method。基本思想：



```python
window_size = 3 # 切分最大单位


def cut(text):
    result = []
    index = 0
    text_length = len(text)
    dic = ['研究', '研究生', '生命', '命', '的', '起源']
    while text_length > index:
        for size in range(window_size + index, index, -1):
            piece = text[index:size]
            if piece in dic:
                index = size - 1
                break
        index = index + 1
        result.append(piece + '----')
    print(result)


if __name__ == '__main__':
    t = '研究生研究生命的起源'
    cut(t)
```

输出：

```python
['研究生----', '研究生----', '命----', '的----', '起源----']
```



### 逆向最大匹配法

Reverse Maximum Match Method，RMM法。原理与MM相同，区别在于切分的方向相反，从被处理文档的末端开始扫描。因汉语的特性，从后向前匹配的精确度更高。

```python
window_size = 3 # 切分最大单位


def cut(text):
    result = []
    index = len(text)
    dic = ['研究', '研究生', '生命', '命', '的', '起源']
    while index > 0:
        for size in range(index - window_size, index):
            piece = text[size:index]
            if piece in dic:
                index = size + 1
                break
        index = index - 1
        result.append(piece + '----')
    print(result)


if __name__ == '__main__':
    t = '研究生研究生命的起源'
    cut(t)

```

输出：

```python
['起源----', '的----', '生命----', '研究----', '研究生----']
```

### 双向最大匹配法

Bi-direction Matching method。将MM和RMM得到的分词结果进行比较，按照最大匹配原则，选词数切分最少的为结果。







REFERENCE:

[1]Python自然语言处理实战-核心技术与算法