---
title: "go重点解析"
date: 2018-07-30T16:27:17+08:00
draft: true
---

## goroutine&channel

go语言对并发的支持是这门语言最重要的特性之一。goroutine很像线程，但是占用内存数少于线程，而且代码量也明显低于线程。

通道是内置的数据结构。为了goroutine之间的通信。