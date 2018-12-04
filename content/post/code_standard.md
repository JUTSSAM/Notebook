---
title: "代码规范"
date: 2018-07-19T11:23:04+08:00
draft: false
---

标签：PHP

---

写代码如写文章，在实现功能的基础上，易读性是不容忽视的，一定要注意注释和必要文档的书写，不光有利于自己和其他工程师的开发，而且有益于同事之间的相处。

# 一般语言代码规范

这里的一般语言指的是常见的常用的编程语言。

## 注释编写

解释关键变量含义以及函数功能，格式符合IDE规范，尽量让在读代码的工程师不用进入到源码函数声明就知道函数的作用。

- 文件属于哪个包那个类
- 包含哪些功能操作
- 注意事项

## 变量命名

除getter，setter , 构造，析构函数等之外，自定义函数名及变量需要注意在他人阅读时的易读性。

- 选词专业、通用
- 动词+名词+介词+名词。表明该函数的工作以及依赖的操作或其他介质

## 为了美观

- 在多行代码均有赋值型操作时，美化代码让每行的等号对齐。
- 等号及逻辑符号前后要加空格。
- 箭头（引用）调用，即->funcName()，如果存在多条，可换行书写。
- 代码中插入其他语言的代码作为字符串，如SQL，遵守该种代码的书写格式。

下面将用PHP代码演示：

## 示例代码片段

```php
<?php
/*
 * This file is part of the {project}
 *
 * (c) {nickname} <{email}>
 *
 * [This source file is subject to the MIT license that is bundled
 * with this source code in the file LICENSE.]
 */
 
 namespace Code\Kernel\Standard;
 
 use PHP\Standard\Psr as JutssamStandard;
 
 /**
 * {ClassName}[ ：{ Introducetion ...}]
 *
 * @author {nickname} <{email}>
 * 
 * @property \Code\Kernel\Comment   $comment
 * @property \Code\Kernel\TextBody  $text_body
 * @property \Code\Kernel\Artistic  $artistic
 */
 
 Class Application extends ServiceProvider
 {
    /**
     * @var array
     */
    protected $item;
    
    /**
     * @var stirng
     */
    protected $type;
    
    /**
     * @var int
     */
    protected $count;
    
    /**
     * Constructor
     *
     * @param array  $item
     * @param int    $count
     * @param string $type
     *
     */
    public function __construct(array $item, int count, string $type)
    {
        parent::__construct();
        
        $this->item  = $item;
        $this->count = $count;
        $this->type  = $type;
    }
    
    /**
     * Getter
     * 
     * @return int
     */
    public function getCount()
    {
        return $this->count;
    }
    
    /** 
     * Push new item in.
     * This function style only can be used in PHP >= 7.0
     * 
     * @param string $item
     * 
     * @return array
     *
     */
    public function push(string $item) : array
    {
        $this->item  = array_push($this->item,$item);
        $this->item  = array_unique($this->item);
        $this->count = count($this->item);
        return $this->item;
    }
    
 }
 
```

## Jetbrains全家桶设置方式

# 数据库代码书写