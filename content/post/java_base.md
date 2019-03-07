---
title: "Java基础知识"
date: 2019-02-26T20:51:47+08:00
draft: false
---

[TOC]

# I

Overview:

1 ) 简单性

2 ) 面向对象

3 ) 分布式

4 ) 健壮性

5 ) 安全性 

6 ) 体系结构中立

7 ) 可移植性

 8 ) 解释型 

9 ) 高性能

10 ) 多线程

11 ) 动态性



String -> Integer

Integer.parseInt(String s);

double/integer -> Stirng

String.valueOf(element);

## 面向对象程序设计

## 反射与代理

## 接口与内部类

## 异常处理

## 泛型程序设计

## 集合框架

## 事件监听器模型

## 并行

## Swing UI

# II

## 流API

## 文件处理与正则表达式

## 数据库

## XML处理

## 注释

## 国际化

## 网络编程

## 高级GUI组件

## 高级图形

## 原生方法



#### List

1 

.add(element) 增加元素

.get(index) 根据索引获取元素

.remove(index) / .remove(element) 根据索引/内容删除元素



2

.contains(element) 是否包含

.set(index,element) 替换

.indexOf(element)  / .lastIndexOf(element) 获取元素索引



3

.subList(fromIndex,toIndex) 截取其中部分连续元素获得新的list

.size() 元素个数

.isEmpty() 返回是否为空



4

.toString() 转换为字符串

.toArray() 转换为数组



#### Map

void clear();

boolean containsKey(Object key);

boolean containsValue(Obejct value);

