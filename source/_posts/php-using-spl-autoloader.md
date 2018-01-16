---
date: 2015/08/11 12:34:50
title: PHP中自动加载的几种实现
categories:
- programming
tags:
- php
---
PHP自动加载是一个很有用的技巧，我们应该在项目中尽量使用autoload来减少维护类加载的工作。 
## 使用__autoload ##
在使用PHP的项目中，如何实现自动加载对于新人总是一个很疑惑的问题，一般写过的一些PHP，或者用过一些框架的同学都知道通过实现__autoload()函数来实现简单的自动加载：

```
function __autoload($classname) {
    $filename = "./". $classname .".php";
    include_once($filename);
}
```
不过这样的方式缺点也能明显就是只能定义一种加载策略，或者把这个函数实现的很复杂来实现多样的自动加载策略。

## 使用spl_autoload_register ##
现在推荐使用的是`spl_autoload_register()`来注册自动加载器，它可以用来注册多个自动加载策略。它的函数签名是:

```
bool spl_autoload_register ([ callable $autoload_function [, bool $throw = true [, bool $prepend = false ]]] )
```
一般来说，我们只关注第一个参数，他是一个callable类型的function，一般简单的传入一个函数名称的字符串，也可以传入类名和方法名的数组，也可以传入一个匿名函数（PHP 5.3之后）。

```
function my_autoloader($class) {
    include 'classes/' . $class . '.class.php';
}

spl_autoload_register('my_autoloader');

// Or, using an anonymous function as of PHP 5.3.0
spl_autoload_register(function ($class) {
    include 'classes/' . $class . '.class.php';
});
```
在PHP 5.3 引入命名空间后，我们还可以使用[一种更方便的自动加载策略](http://php.net/manual/en/function.spl-autoload-register.php#92514)

```
spl_autoload_extensions(".php"); // comma-separated list
spl_autoload_register();

// 等价于
function my_autoload ($pClassName) {
    include(__DIR__ . "/" . $pClassName . ".php");
}
spl_autoload_register("my_autoload");
```
据说第一种可以效率更高。

## 使用SplClassLoader ##
SplClassLoader是Jonathan H. Wage，Roman S. Borschel等人开发的一个用于PHP 5.3根据命名空间和类名实现自动加载的类。使用比较方便，这里推荐使用。其实现并不复杂使用一个类，源码如下（去掉了注释）:

```
class SplClassLoader
{
    private $_fileExtension = '.php';
    private $_namespace;
    private $_includePath;
    private $_namespaceSeparator = '\\';

    public function __construct($ns = null, $includePath = null) {
        $this->_namespace = $ns;
        $this->_includePath = $includePath;
    }
    public function setNamespaceSeparator($sep) {
        $this->_namespaceSeparator = $sep;
    }
    public function getNamespaceSeparator() {
        return $this->_namespaceSeparator;
    }
    public function setIncludePath($includePath) {
        $this->_includePath = $includePath;
    }
    public function getIncludePath() {
        return $this->_includePath;
    }
    public function setFileExtension($fileExtension) {
        $this->_fileExtension = $fileExtension;
    }
    public function getFileExtension() {
        return $this->_fileExtension;
    }
    public function register() {
        spl_autoload_register(array($this, 'loadClass'));
    }
    public function unregister() {
        spl_autoload_unregister(array($this, 'loadClass'));
    }
    public function loadClass($className) {
        if (null === $this->_namespace || $this->_namespace.$this->_namespaceSeparator === substr($className, 0, strlen($this->_namespace.$this->_namespaceSeparator))) {
            $fileName = '';
            $namespace = '';
            if (false !== ($lastNsPos = strripos($className, $this->_namespaceSeparator))) {
                $namespace = substr($className, 0, $lastNsPos);
                $className = substr($className, $lastNsPos + 1);
                $fileName = str_replace($this->_namespaceSeparator, DIRECTORY_SEPARATOR, $namespace) . DIRECTORY_SEPARATOR;
            }
            $fileName .= str_replace('_', DIRECTORY_SEPARATOR, $className) . $this->_fileExtension;

            // @add wuxu 验证文件是否存在，要不可能会抛出异常
            $fileName = ($this->_includePath !== null ? $this->_includePath . DIRECTORY_SEPARATOR : '') . $fileName;

            if (file_exists($fileName)) require $fileName;
        }
    }
}
```
完整的代码见 [GitHub Gist](https://gist.github.com/jwage/221634 "https://gist.github.com/jwage/221634")。
（还有一些在此基础上的修改版本，比如：https://gist.github.com/jwage/221634）

它的使用方式如下：

```
 // Example which loads classes for the Doctrine Common package in the
 // Doctrine\Common namespace.
 $classLoader = new SplClassLoader('Doctrine\Common', '/path/to/doctrine');
 $classLoader->register();
```
这是一个实现使用命名空间的PHP项目的自动加载的方式（PSR-0），使用上面的注册代码，可以注册一个Doctrine\Common的namespace，它对应的根目录是/path/to/doctrine。

## 总结 ##
这样对于实现一个使用命名空间的项目非常重要。
接下来会写一篇如何在一个项目中使用namespace（PSR-0）来使项目的结构更加科学。
