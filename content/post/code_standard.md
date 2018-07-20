---
title: "代码规范"
date: 2018-07-19T11:23:04+08:00
draft: false
---

标签：PHP

---

看PSR文档寻找规范太繁琐，不如自己写一份示例

## 示例代码片段

```
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

## PHPStorm设置方式