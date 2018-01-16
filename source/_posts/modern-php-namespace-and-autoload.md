---
title: Modern PHP 命名空间、traits等
date: 2016-06-10 13:21:49
categories:
- programming
tags:
- php
---

## 命名空间
按照PSR的标准，应该是一个文件对应一个类，并且命名空间与文件系统路径一一对应，这样便于理解。但是也允许一个文件有多各类，甚至一个文件有多分命名空间，一个命名空间也可以分散在不同的文件/文件夹中，这都是允许的，但是不是推荐的做法。

因为命名空间和文件系统的对应关系是通过自动加载（auto_load）定义的，只要定义的spl加载器能找到要引用的类的实际文件就可以正常运行，但是最好不要使用违反PSR标准的命名空间。这不利于源代码的管理和调试。

## 面向接口编程

接口是两个对象的共同遵循的规约，它使得对象不在依赖于另一个对象是什么，而只依赖于它能做什么。面向接口使得我们能使用那些实现了某种接口的第三方代码。这样我们不需要知道第三方代码是怎么实现的，只要知道它们能做那些事就可以了。

## Traits
 Traits是PHP 5.4 引入的新特性。最初看到这个概念的时候觉得有点困惑，因为其他的语言基本没有traits的概念（Ruby有相似的概念）。Traits用来实现更细化的代码复用，它的行为更像类，但是看起来很像接口，但它并不是类或者接口。

Traits可以看作是类的部分实现（partial class implementaion），它能被用于多个PHP类中。在类的内部使用 `use` 关键字引入traits即可以把traits实现的功能加入该类中。

traits用来解决经典单继承模型难以解决的问题：两个不存在关联的类共享相似的行为。通过复杂的类型设计或者模式可以实现，但是这样的类型系统通常要非常复杂不利于理解，因为要为两个不相关的类设计一个本不存在的祖先类，而traits可以非常方便的在两个不相关的类中共享实现：通过把trait引入类中就可以了。

traits的使用和类很相似，通过trait关键字定义：

```
trait GeoTraits {
	protected $lnt;

	public function getLnt() {
		...
	}
	// trait implementation 
}
```
直接在类中使用traits:

```
class GeoClass {
	use GeoTraits;
	// other parts of this class
}
```
traits、类和接口都通过use关键字引入，不过traits是在类的内部引入，作为类实现的一部分使用的。

tratis的使用更像是一个预处理过程，PHP解释器会在编译期直接复制traits的内容到类中，解释器并不能保证这一行为不会导致代码的不兼容。所以要注意，如果在traits中使用了不是在trait中定义的属性或方法（即是定义在使用trait的类中的属性或方法）时，要保证这些属性或者方法是存在的。

## Generators

Generator是PHP5.5引入的特性，到现在还很少被使用。Generator可以看作一个简单的遍历器（iterator）。generator不需要实现Iterator接口，而是在需要时计算、产生（yield）遍历的值。PHP的遍历器是实现计算好的数据集，而generator是在使用时返回值的，对于大量的数据集这样可以节省很多开销。

注意generator是一个只能遍历一遍的单向遍历器，不能回退，不能直接定位到某个元素。generator就是一个使用了`yield`关键字的普通函数。这个函数可以使用一次或者多次`yield`。

```
function simpleGenerator() {
	yield "val1";
	yield "val2";
	yield "val3";
}

foreach simpleGenerator() as $val {
	echo $val;
}
```
调用一个generator函数会返回一个Generator类的对象，这个对象能使用`foreach`遍历，在每一次遍历中，generator会计算下一个值，因为是在请求值时才计算下一个返回的值，所以在处理很大的数据集时，generator并不需要使用很多的内存。比如：

```
function rangeGenerator($len) {
	for ($i=0; i<$len; $i++) {
		yield $i;
	}
}

foreach (rangeGenerator(1000000) as $i) {
	echo $i, PHP_EOF;
}
```
这里就并不需要分配一个 1000000 大小的数组，generator也可以用来读取大文件，而不是把整个文件读入内存中，下面是读取一个csv文件的示例：

```
function getRows( $file) {
	$handle = fopen( $file, ' rb' ) ;
	if ( $handle === false) {
		throw new Exception( ) ;
	}
	while ( feof( $handle) === false) {
		yield fgetcsv( $handle) ;
	}
	fclose( $handle) ;
}

foreach ( getRows( ' data. csv' ) as $row) {
	print_r( $row) ;
}
```
其中`fgetcsv`是php提供的函数。

在有些时候generator能简化一些任务，当然不实用generator也可以实现所有的功能。

## Closure/闭包
闭包和匿名函数都是PHP 5.3中引入的。然而很多开发者现在仍很少使用php的闭包和匿名函数。可能这些特性看起来就比较复杂吧。

php的闭包和js的闭包很相似，可以用相同的思路去理解。匿名函数其实是比较常见的，匿名函数可以被赋给一个变量，或者像一个对象一个传递。匿名函数常用来作为方法的回调。

闭包和匿名函数本理论上是两个不同的东西，但是PHP中把它们混为一谈了。PHP的闭包和匿名函数使用相同的语法，它们实际上是*伪装*为函数的对象，它们都是 `Closure` 类的实例， 闭包是一等值类型，就和 String和Integer一样。

> PHP变量后面使用 () 时，如 `$var()` 这样的调用会尝试调用 `__invoke()` 魔术方法。

前面说过，匿名函数常作为方法的回调参数，如 `array_map` 和 `preg_replace_callback()`。

PHP中的闭包使用和JS有很大不同，虽然都是闭包的理论。PHP的闭包需要使用 `bindTo()` 方法或者 `use` 关键字使用。PHP的闭包有一个状态附着（attach state）的概念，也就是必须手动将状态/变量（state）附着到一个闭包对象上（使用闭包的`bindTo()`方法）。一种更常用的方法是使用`use`关键字。

```
function enclosePerson($name) {
	return function ($doComande) use ($name) {
		return sprintf('%s %s', $name, $doCommand);
	};
}

$clay = enclosePerson('Clay');
echo $clay('get me sweet tea');
```
从上面可以看出，需要手动将变量绑定到闭包中才可以在闭包内使用外部的变量。这和JS很不一样。上面的函数返回的是一个 Closure 对象实例，所以 `$clay` 有自己的属性和方法，常用的如 `__invoke()` 和 `bindTo()` 方法。 `bindTo`方法有一些特殊的用法，如可以讲一个闭包对象绑定到另外的对象上。这样闭包可以访问该对象的成员对象（包括protected和private）的。

在PHP框架中的路由映射中常用`bindTo`方法，将匿名的路由回调函数绑定到应用对象（application object）上。

## OPcache
从PHP 5.5开始集成了字节码缓存，那就是Zend OPcache。字节码缓存会存储预先编译的PHP字节码，这意味着解释器不需要在每次请求的时候都去读取、解析和编译PHP代码，而是直接读取预编译的字节码直接执行，这可以节省很多时间开销，提升性能。

旧的PHP默认没有开启OPcache，使用 `--enable-opcache`编译选项开启，然后在php.ini配置中，添加扩展： `zend_extension=/path/to/opcache.so`。注意如果需要使用xdebug的话，需要将opcache在xdebug之前加载。

OPcache的常用配置项包括

```
# 在开发环境这个选项设置为1，生产环境可以设置为0
opcache.validate_timestamps=1
opcache.revalidate_freq=0
```

## 内置Web Server
> php -S localhost:4000

在项目目录使用上面的命令，可以启动内置的服务器

