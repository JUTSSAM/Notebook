---
title: "正则表达式"
date: 2019-03-08T14:59:39+08:00
draft: false
---

Python

RE

re.search(patern,string)

re.findall(patern,string)

|符号|含义|例|
| :--------: | :-------------: | :---:|
|.|匹配任意一个字符|"a.c"->"abc","branch"|
|^|匹配开始的字符串|"^a"=>所有以a开头的字符串|
|$|匹配结尾的字符串|"^a"=>所有以a结尾的字符串|
|[]|匹配多个字符|[bcr]at=>"bat" "rat" "cat"|

Numpy

ndarray

初始化：

vector = np.array([1,2,3,4])
matrix = np.array([[1,'Tim'],[2,'John']])

获取数组维度：vector.shape =>  (4,)

获得数组大小：matrix.size => 2

数据比较：

m = (matrix == 25) 返回布尔数组

数据替换：matrix[m] = 10 将数据中的25换为10

统计计算：

sum() mean() max()

矩阵需要置顶行或者列