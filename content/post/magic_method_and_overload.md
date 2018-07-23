---
title: "魔术方法和重载"
date: 2018-07-19T11:28:05+08:00
draft: false
---

标签： PHP

---
![PHP中魔术方法][1]
## 魔术方法简要叙述
### \_\_construct&\_\_destruct
__构造函数__与__析构函数__，这两个函数在许多编程语言中均有实现。
PHP中比较特殊的是，需要以双下划线开头，(>=5.6)。并且不能有重名函数存在。
### \_\_get&\_\_set&\_\_isset&\_\_unset
在访问不可访问属性时将会触发此类函数,原本的操作反而不会执行。此类函数可以用来实现类的getter与setter，但是对于PHP这种效率为王的语言，开发者们多半不会考虑此类繁琐的事情。
实现代码如下：

```
<?php
	class Magic{
		
		protected $pro1;
		protected $pro2;

		function __get($pro){
			echo "__get is running and is getting {$pro}\n";
			return $this->$pro;
		}

		function __set($pro,$value)
		{
			echo "__set is running and is seting {$pro} as {$value}\n";
			$this->$pro = $value;
		}

		function __isset($pro)
		{
			return isset($this->$pro);
			echo "__isset is running and is checking {$pro}.\n";
		}

		function __unset($pro)
		{
			unset($this->$pro);
			echo "__unset is running and is checking {$pro}.\n";
		}
	}

	$magic = new Magic();

	$magic->pro2 = '2'; //赋值没有执行
	echo $magic->pro2."\n";  //同样echo没有运行

	$re = isset($magic->pro2);
	var_dump($re);

	unset($magic->pro2);
		
	
```

输出:
	
```
__set is running and is seting pro2 as 2
__get is running and is getting pro2
2

bool(true)

__unset is running and is checking pro2.

```

### \_\_call&\_\_callstatic

```
<?php
	class Magic{
		
		protected $pro1;
		protected $pro2;

		protected function func1()
		{
			var_dump($this->pro1);
		}

		public static function func2(){
			echo "func2 is running.\n";
		}

		function __call($foo1,$arg){
			echo "program is calling {$foo1} using ".json_encode($arg)."\n";	
		}

		public static function __callStatic($foo1,$arg){
			echo 'program is staticly calling  {$foo1} using '.json_encode($arg)."\n";
		}
	}

	$magic = new Magic();
	$magic->fun1(1,1);
	Magic::func2(2,2);
	Magic::func3(3,3);
```
输出：
```
program is calling fun1 using [1,1]
func2 is running.
program is staticly calling  {$foo1} using [3,3]
```

## 魔术方法底层实现原理



  [1]: https://upload-images.jianshu.io/upload_images/1594723-5813e1eb121f6be6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/2400