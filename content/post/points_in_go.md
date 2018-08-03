---
title: "go部分重点记录"
date: 2018-07-30T10:06:05+08:00
draft: false
---


## go语言的字符(串)
_Strings, bytes, runes and characters in Go_

首先搞清楚字符串的储存方法，对于一个字符串从bytes角度来看是一视同仁的，不管你存的是什么字符、能不能输出，都是以二进制字节为单位存着。

ASCII：固定8bit

UTF8：变长，可用1-4个字节表示一个符号，根据不同的符号变化字节长度。

go语言中，源代码存储格式是UTF8，因此字符串字面量(string literals)一般也是UTF8编码

在Unicode中，将一个character定义为一个code unit，在go中对照的有rune类型。在range方法遍历字符串数组时，会按照变长类型区分字符。（汉字占3字节，数字英文占一字节）

可以使用package unicode/utf8 对字符串进行操作，
如：
	fmt.Println(utf8.RuneCountInString(s))
可获取rune类型字符数。

字符串输出:

| 符号 | 输出 | exmple |
|----|----|----|
| %x | 单个字节16进制表示 | fmt.Printf("%x",sample) |
| %q | 无法打印字符用单个字节16进制表示 | fmt.Printf("%q",sample) |
| %+q | 无法打印字符用单个字节16进制表示，非ASCII码打印其Unicode编码 | fmt.Printf("%+q",sample) |

[strings of go](https://blog.golang.org/strings)

[dive-into-go-1](http://colobu.com/2016/06/15/dive-into-go-1/)

## 数组与切片
数组：有限的同一元素类型的对象的序列。定长。
Slice：描述数组的一个连续的片段。_[)_

```
type SliceHeader struct{
	Data uintptr
	Len  int
	Cap  int
}
```

数组可能在数组和多个slice中公用，会导致修改异步问题。

## 数据类型转换


## 结构体匿名字段

## 鸭子类型
_duck typing_

go中通过接口和duck typing支持面向对象的编程。

如果一个接口有一系列方法，不需要implement接口就可以使用这些方法。(If it has a set of methods that match an interface, then you can use it wherever that interface is needed without explicitly defining that your types implement that interface.)

[Duck typing in Go](https://medium.com/@matryer/golang-advent-calendar-day-one-duck-typing-a513aaed544d)

↑一篇文章看懂duck typing