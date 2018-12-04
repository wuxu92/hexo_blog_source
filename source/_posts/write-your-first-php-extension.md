---
date: 2015/08/18 12:34:50
title: 编写你的第一个PHP扩展模块(一)
categories:
- programming
tags:
- php
- extension
---
一直以来对于PHP的扩展编写都比较感兴趣但是却没有什么更深的接触，正好最近要研究一下，做个记录吧。

参考： 
[http://www.laruence.com/2009/04/28/719.html](http://www.laruence.com/2009/04/28/719.html "http://www.laruence.com/2009/04/28/719.html")
[http://www.walu.cc/phpbook/5.md](http://www.walu.cc/phpbook/5.md "http://www.walu.cc/phpbook/5.md")

首先我们从头安装一个PHP环境，现在最新的稳定版的PHP是5.6.12。

## 编译安装php ##
以下在Linux（centos）下完成

1. 下载src： `wget http://cn2.php.net/get/php-5.6.12.tar.bz2/from/this/mirror` wget 下载的文件可能被命名为mirror，需要重命名一下 `mv mirror php-5.6.12.tar.bz2`
2. 解压： `tar xjvf php-5.6.12.tar.bz2`
3. `cd php-5.6.12` 编译安装： `./configure --prefix=/home/wuxu/data/php5.6/ --enable-debug --enable-maintainer-zts`
4. `make`
5. `make install`
6. `make clean`

这样一个新的php安装在了家目录下（/home/wuxu/data/php5.6）。

## 扩展的准备工作 ##
首先要理解为PHP内核编写的扩展的两个工作方式，一种是编译为动态共享对象/可装载模块，也就是常见的.so扩展，这种扩展可以在php的配置文件中方便的开启或者关闭；另外一种方式是静态编译到PHP中，使用静态方法编译比较容易上手，鸟哥的文章中也是使用的静态编译方式，所以我们也使用静态编译方式来练习。

PHP在源码中提供了一个扩展骨架构造脚本： ext_skel，脚本放在php-5.6.12/ext目录下。它的使用方式如下： 

```php
./ext_skel --extname=mfs --proto=mfs.def
```
解释一下，--extname明显就是我们要创建的扩展的名称，--proto的proto是prototype的缩写，也就是扩展对外提供的函数原型，可以在这个文件中添加要导出的函数签名，每个函数做一行，这样ext_skel脚本可以自动创建它们的骨架代码。比如鸟哥的例子中的字符串复制函数:

```
string self_concat(string str, int n)
```
把这一行保存为mfs.def文件，放在ext文件夹下。

## 基本骨架 ##
运行上面的ext_skel命令，就会在ext文件夹下创建一个mfs的文件夹，并声称一些代码文件和配置文件。 php扩展在Liux下的配置文件是 ext/mfs/config.m4；m4有自己的语法，不过我们并不需要熟悉它，只需要简单去掉一些注释就可以了。打开配置文件config.m4；大概在16行和18行，找到`PHP_ARG_ENABLE(mfs, whether to enable mfs support` 相关的内容，这一行是用来重新生成configure文件时起作用的，取消这一行及
它后面的第二行`[ --enable-myfunctions                Include myfunctions support])`，中间有一行不要取消注释。这样就可以重新生成configure文件可以使用enable-mfs来静态编译扩展。

完成上面的工作后，重新生成configure文件并编译安装php。

```
cd /path/to/php-5.6.12
./buildconf --force
./configure --enable-fms --prefix=/home/wuxu/data/php5.6 --with-config-file-path=/home/wuxu/data/etc/php.ini
make
make install
```
这样编译的php就带有了fms扩展，并可以使用在def文件中定义的原型函数。但是由于并没有编写那个函数的具体内容，所以这个是str_concat函数并不能起作用，不过ext_skel脚本还导出了一个函数： confirm_mfs_compiled(str); 可以用下面的脚本检测fms扩展是否可用了：

```php
<?php
print confirm_mfs_compiled("myextension");
?>

// output: 
// "Congratulations! You have successfully modified ext/mfs
//  
// config.m4. Module mfs is now compiled into PHP."
```
表明脚本已经编译到php了。下面我们就可以开始编写self_concat的具体内容了。

## 起始代码 ##
先看一下有ext_skel脚本自动生成的self_concat函数代码，在ext/mfs/mfs.c:76 

```
PHP_FUNCTION(self_concat)
{
	char *str = NULL;
	int argc = ZEND_NUM_ARGS();
	int str_len;
	long n;
	if (zend_parse_parameters(argc TSRMLS_CC, "sl", &str, &str_len, &n) == FAILURE)
		return;
	php_error(E_WARNING, "self_concat: not yet implemented");
}
```
这些代码是一个到处函数的基本框架了。
使用PHPFUNCTION()宏来生成一个适合于Zend引擎的函数原型。其中比较重要的是 zend_parse_parameters 函数，用来获取函数传递的参数；该函数的原型是：

```c
zend_parse_parameters(int num_args TSRMLS_DC, char *type_spec, …);
```
第一个参数是参数的个数，通常使用`ZEND_NUM_ARGS()`的返回值，TSRMLS_DC这个照写就可以了，第三个参数比较复杂是一个表示函数期望的参数类型的字符串，后面紧跟参数值的变量列表，这里有一个PHP的松散变量和动态类型推断到C语言类型的转换。

第三个参数根据一个参数类型对照表生成:


|类型指定符 |对应的C类型 | 描述|
|------------|-----------|------|
|l |long | 符号整数|
|d |double | 浮点数|
|s |char *, int | 二进制字符串，长度|
|b |zend_bool | 逻辑型（1或0）|
|r |zval * | 资源（文件指针，数据库连接等）|
|a |zval * | 联合数组|
|o |zval * | 任何类型的对象|
|O |zval * | 指定类型的对象。需要提供目标对象的类类型|
|z |zval * | 无任何操作的zval|

考虑上面代码的实例：

```
if (zend_parse_parameters(argc TSRMLS_CC, "sl", &str, &str_len, &n) == FAILURE)
	return;
```
"sl": 第一个字符s代表二进制字符串，它在后面的参数列表中对应两个值，一个 &str， 一个&strlen；第二个字符'l'（L的小写）,表示整数类型参数对应 &n。

> 扩展中的字符串都是二进制字符串，即并不以\0作为字符串结束，而是使用一个str_len表示字符串长度，具体看_zval结构体。

下面的工作就是修改这个函数了。

## 完成第一个导出函数 ##

将自动生成的函数更新为：

```
PHP_FUNCTION(self_concat)
{
	char *str = NULL;
	int argc = ZEND_NUM_ARGS();
	int str_len;
	long n;
	char *result; /* Points to resulting string */
	char *ptr; /* Points at the next location we want to copy to */
	int result_length; /* Length of resulting string */
	
	if (zend_parse_parameters(argc TSRMLS_CC, "sl", &str, &str_len, &n) == FAILURE) {
		return;
	}

	result_length = (str_len * n);
	result = (char *) emalloc(result_length + 1);
	ptr = result;
	while (n--) {
		memcpy(ptr, str, str_len);
		ptr += str_len;
	}

	*ptr = '\0';

	RETURN_STRINGL(result, result_length, 0);
} 
```
按照上面的方法，重新编译php， 就可以在php文件中直接使用slef_concat()函数拼接字符串了。

```
<?php
print self_concat("pop_", 10);
```
保存为confirm.php;运行：

```
php confirm.php
```
会输出拼接的字符串了。

## 待续 ##


