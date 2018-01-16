---
date: 2015/08/10 12:34:50
title: PHP变量在内核中的实现
categories:
- programming
tags:
- php
- inside
---

PHP本身是一种弱类型的语言，可以在程序中改变变量存储的值的类型。那么这些变量在PHP的底层是如何实现的呢，理解内核变中变量的实现机制将有利于我们理解PHP的变量系统。

## 变量的类型 ##
对PHP有一定理解的同学应该都已经知道了PHP在内核中是使用zval这个结构体来存储变量的，也就是说不同的变量在底层都是一个zval结构体，它定义在Zend/zend.h中：

```
struct _zval_struct {
    zvalue_value value; /* 变量的值 */
    zend_uint refcount__gc;
    zend_uchar type;    /* 变量当前的数据类型 */
    zend_uchar is_ref__gc;
};
typedef struct _zval_struct zval;

//在Zend/zend_types.h里定义的：
typedef unsigned int zend_uint;
typedef unsigned char zend_uchar;
```

其中保存变量的值的value则是zvalue_value类型，它是一个union：

```
typedef union _zvalue_value {
    long lval;  /* long value */
    double dval;    /* double value */
    struct {
        char *val;
        int len;
    } str;
    HashTable *ht;  /* hash table value */
    zend_object_value obj;
} zvalue_value;
```
PHP使用zvalue_value实现8种数据类型，这些类型在内核中分别对应特定的常量。通过这个union可以理解常用的类型判断函数： `is_null, is_bool, is_long, is_double, is_string, is_array, is_object, is_resource` 他们的效率应该是很高的。同时也可以猜想gettype函数的实现大概使用了Z_TYPE_P宏，这个宏大概和zval结构的type有关
在Zend/zend_operators.h中定义的宏：

```
#define Z_TYPE(zval)        (zval).type
#define Z_TYPE_P(zval_p)    Z_TYPE(*zval_p)
#define Z_TYPE_PP(zval_pp)  Z_TYPE(**zval_pp)
```

> 注意，虽然在32位系统中，signed long的存储范围是-2147483648～2147483647 的整数，但是要注意，如果整数变量超出了这个范围并不会直接溢出，而是会转换程double类型继续计算。

PHP内核还同时在zval结构里保存着字符串的实际长度， 这个设计使PHP可以在字符串中嵌入‘\0’字符，也使PHP的字符串是二进制安全的， 可以安全的存储二进制数据

## 变量的值 ##
string型变量比较特殊，因为内核在保存String型变量时，不仅保存了字符串的值，还保存了它的长度， 所以它有对应的两种宏组合STRVAL和STRLEN，即：Z_STRVAL、Z_STRVAL_P、Z_STRVAL_PP与Z_STRLEN、Z_STRLEN_P、Z_STRLEN_PP。 前一种宏返回的是char *型，即字符串的地址；后一种返回的是int型，即字符串的长度。

Array型变量的值其实是存储在C语言实现的HashTable中的， 我们可以用ARRVAL组合宏（Z_ARRVAL, Z_ARRVAL_P, Z_ARRVAL_PP）这三个宏来访问数组的值。

对象是一个复杂的结构体（zend_object_value结构体），不仅存储属性的定义、属性的值，还存储着访问权限、方法等信息。 内核中定义了以下组合宏让我们方便的操作对象： OBJ_HANDLE：返回handle标识符， OBJ_HT：handle表， OBJCE：类定义， OBJPROP：HashTable的属性， OBJ_HANDLER：在OBJ_HT中操作一个特殊的handler方法。

资源型变量的值其实就是一个整数，可以用RESVAL组合宏来访问它，我们把它的值传给zend_fetch_resource函数，便可以得到这个资源的操作句柄，如mysql的链接句柄等。

## 创建PHP变量 ##
在内核中是如何创建zval的呢，PHP内核中提供了一个MAKE_STD_ZVAL(pzv)宏，它使用内核的方式类申请一块内存，并将其地址赋给pzv。这个宏能自动处理内存不足的问题。
获取空间后，就可以给这个zval赋值了。旧的做法是先确定zval的类型：`Z_TYPE_P(pzv)=ISNULL` 来设置其为null类型，再通过Z_SOMEVAL_P的宏类赋值：

```
Z_TYPE_P(pzv)=IS_BOOL;
Z_BVAL(PZV)=1;
```
不过现在PHP内核提供了更多的宏可以更方便的操作zval的值。

新宏	其它宏的实现方法

- ZVAL_NULL(pvz); **(注意这个Z和VAL之间没有下划线！)**	`Z_TYPE_P(pzv) = IS_NULL;`**(IS_NULL型不用赋值，因为这个类型只有一个值就是null)**
- ZVAL_BOOL(pzv, b); **(将pzv所指的zval设置为IS_BOOL类型，值是b)**	`Z_TYPE_P(pzv) = IS_BOOL; Z_BVAL_P(pzv) = b ? 1 : 0;`
- ZVAL_TRUE(pzv); **(将pzv所指的zval设置为IS_BOOL类型，值是true)**	ZVAL_BOOL(pzv, 1);
- ZVAL_FALSE(pzv); **(将pzv所指的zval设置为IS_BOOL类型，值是false)**	ZVAL_BOOL(pzv, 0);
- ZVAL_LONG(pzv, l); **(将pzv所指的zval设置为IS_LONG类型，值是l)**	`Z_TYPE_P(pzv) = IS_LONG;Z_LVAL_P(pzv) = l;`
- ZVAL_DOUBLE(pzv, d); **(将pzv所指的zval设置为IS_DOUBLE类型，值是d)**	`Z_TYPE_P(pzv) = IS_DOUBLE; Z_DVAL_P(pzv) = d;`
- ZVAL_STRINGL(pzv,str,len,dup);**(下面单独解释)**	`Z_TYPE_P(pzv) = IS_STRING;Z_STRLEN_P(pzv) = len;if (dup) {Z_STRVAL_P(pzv) =estrndup(str, len + 1);} else {Z_STRVAL_P(pzv) = str;}`
- ZVAL_STRING(pzv, str, dup);	`ZVAL_STRINGL(pzv, str,strlen(str), dup);`
- ZVAL_RESOURCE(pzv, res);	`Z_TYPE_P(pzv) = IS_RESOURCE;`
- Z_RESVAL_P(pzv) = res;

## 变量的存储方式 ##
用户在PHP中定义的变量可以在一个HashTable中找到，当PHP中定义了一个变量，内核会自动把它的信息存储到一个用HashTable实现的符号表里。
全局作用域的符号表在调用扩展的RINIT方法前创建，并且在RSHUTDOWN方法执行之后自动销毁。

一个例子

```
<?php
$foo = 'bar';
?>
```

上面是一段PHP语言的例子，我们创建了一个变量，并把它的值设置为'bar'，在以后的代码中我们便可以使用$foo变量。相同的功能我们怎样在内核中实现呢？我们可以先构思一下步骤：

- 创建一个zval结构，并设置其类型。
- 设置值为'bar'。
- 将其加入当前作用域的符号表，只有这样用户才能在PHP里使用这个变量。


具体的代码为：

```
{
    zval *fooval;

    MAKE_STD_ZVAL(fooval);
    ZVAL_STRING(fooval, "bar", 1);
    ZEND_SET_SYMBOL( EG(active_symbol_table) ,  "foo" , fooval);
}
```
首先，我们声明一个zval指针，并申请一块内存。然后通过ZVAL_STRING宏将值设置为‘bar’,最后一行的作用就是将这个zval加入到当前的符号表里去，并将其label定义成foo，这样用户就可以在代码里通过$foo来使用它了。

## 变量的检索 ##
在PHP中定义的变量，在内核中通过zend_hash_find()函数来找到当前作用域下用户定义好的变量。zend_hash_find是内核提供的操作hashTable的API之一。
[http://www.walu.cc/phpbook/2.5.md](http://www.walu.cc/phpbook/2.5.md "http://www.walu.cc/phpbook/2.5.md")

## 类型转换 ##
我们可以通过符号表获取用户在PHP中定义的变量了，想想一下在PHP中的自动类型转换，它在底层C中是怎么实现的呢。
内核中提供了很多函数专门来帮助实现类型转换，这类函数统一的形式为： `convert_to_*()`

```
//将任意类型的zval转换成字符串
void change_zval_to_string(zval *value)
{
    convert_to_string(value);
}

//其它基本的类型转换函数
ZEND_API void convert_to_long(zval *op);
ZEND_API void convert_to_double(zval *op);
ZEND_API void convert_to_null(zval *op);
ZEND_API void convert_to_boolean(zval *op);
ZEND_API void convert_to_array(zval *op);
ZEND_API void convert_to_object(zval *op);

ZEND_API void _convert_to_string(zval *op ZEND_FILE_LINE_DC);
#define convert_to_string(op) if ((op)->type != IS_STRING) { _convert_to_string((op) ZEND_FILE_LINE_CC); }
```
其中，convert_to_string其实是一个宏函数，调用的另外一个函数；另外没有convert_to_resource()的转换函数，因为资源的值在用户层面上，根本就没有意义，内核不会对它的值(不是指那个数字)进行转换。

参考： [PHP变量在内核中的实现](http://www.walu.cc/phpbook/2.md "http://www.walu.cc/phpbook/2.md")
