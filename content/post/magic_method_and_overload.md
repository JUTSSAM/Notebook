---
title: "魔术方法和重载"
date: 2018-07-19T11:28:05+08:00
draft: false
---

标签： PHP

---
![PHP中魔术方法][1]
新标签中打开可查看大图

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
在对象中调用一个不可访问方法时，\_\_call()会被调用。
静态调用一个不可访问方法时，\_\_callStatic()会被调用。
这两个函数均有两个参数：string $name -- 函数名，array $args -- 调用参数数组。
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
			echo "program is calling {$foo1} with ".json_encode($arg)."\n";	
		}

		public static function __callStatic($foo1,$arg){
			echo 'program is staticly calling  {$foo1} with '.json_encode($arg)."\n";
		}
	}

	$magic = new Magic();
	$magic->fun1(1,1);
	Magic::func2(2,2);
	Magic::func3(3,3);
```
输出：
```
program is calling fun1 with [1,1]
func2 is running.
program is staticly calling  {$foo1} with [3,3]
```
在许多面向对象的语言当中，均有重载的概念，重载(overload)即函数/方法有相同的名称，但是参数列表不同，这些函数互相称为重载函数/方法，在调用时根据参数不同来区别不同的函数。

例如在Java中，类中可以创建多个方法，它们具有相同的名字，但具有不同的参数和定义。

在官方文档中给出的用法是上述两个函数可以用作函数的重载，但是用于重载反而没有充分利用到该函数的特性。

在开源项目[EasyWecaht](https://github.com/overtrue/wechat)中有这样一段代码：
```
class Factory
{
    /**
     * @param string $name
     * @param array  $config
     *
     * @return \EasyWeChat\Kernel\ServiceContainer
     */
    public static function make($name, array $config)
    {
        $namespace = Kernel\Support\Str::studly($name);
        $application = "\\EasyWeChat\\{$namespace}\\Application";
        return new $application($config);
    }
    /**
     * Dynamically pass methods to the application.
     *
     * @param string $name
     * @param array  $arguments
     *
     * @return mixed
     */
    public static function __callStatic($name, $arguments)
    {
        return self::make($name, ...$arguments);
    }
}
```
其使用方法是$app = Factory::$objName，利用这个函数可以做一个框架入口，具有很好的封装效果。

### 其余函数
剩下的函数不是本文叙述的重点，故放入表格简单概括。

| function      | trigger  | feature  |
| :--------:    | :-----:  | :----:  |
| __clone()     | $obj2 = clone $obj1; |   浅复制    |
| __invoke()    |   以使用函数的方法使用对象   |   -   |
| __toString()  |   在如echo printf等输出类函数中或者其他需要字符串的情况下，对象作为参数时，会调用对象内的本函数   |  -  |
| __wakeup()    | unserialize()| 工作前的准备 |
| __sleep()     | serialize()  | 同上 |
| __set_state() | ver_export() | debug |
| __debugInfo() | var_dump()   | debug |

## 魔术方法底层实现原理



  [1]: https://upload-images.jianshu.io/upload_images/1594723-5813e1eb121f6be6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/2400