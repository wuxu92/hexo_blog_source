---
date: 2015/08/17 12:34:50
title: 使用后期静态绑定实现MVC的Model的基类
categories:
- programming
tags:
- php
- tricks
---

PHP 5.3中引入了一个后期静态绑定的功能，对于静态绑定都比较熟悉了，就是static修饰的方法或者字段，那这个后期静态绑定又是干什么的呢？

如果在一个类中定义一个静态方法或者静态域成员，如下：

```php
class Person {
	public static $type = 'man';
	public static function run() {
		echo 'run';
	}
}
```
以对于静态成员的理解，这些static修饰的成员在所有对象中共享，只有一个实例，可以使用`Person::run()` 这样的方式直接调用而不需要实例化类。

## self调用 ##
如果在类的内部直接调用这些静态方法，我们一般使用self来指代当前类:

```php
class Person {
	...

	public staic function eat() {
		self::run(); // run before eat...
		// do some eating here
	}
}
```
这里使用self来指代当前的类，并用于调用其静态方法。这里，`self::`和`__CLASS__` 是一样的，都是**对当前类的静态引用**

## 后期静态绑定 ##

常用示例代码：

```php
<?php
class A {
    public static function who() {
        echo __CLASS__;
    }
    public static function test() {
        self::who();
    }
}

class B extends A {
    public static function who() {
        echo __CLASS__;
    }
}

B::test();
?>
```

> 后期静态绑定本想通过引入一个新的关键字表示运行时最初调用的类来绕过限制。简单地说，这个关键字能够让你在上述例子中调用 test() 时引用的类是 B 而不是 A。最终决定不引入新的关键字，而是使用已经预留的 static 关键字。

这里简单的理解就是，B是A的子类，有时候在使用B调用A中实现的的静态方法时，需要使用子类B的一些静态域实现，而不是A实现的静态域。使用`self::` 调用静态成员只会调用A类的静态成员。这是后应该使用后期静态绑定关键字 `static`来代替`self`

```
<?php
class A {
    public static function who() {
        echo __CLASS__;
    }
    public static function test() {
        static::who(); // 后期静态绑定从这里开始
    }
}

class B extends A {
    public static function who() {
        echo __CLASS__;
    }
}

B::test(); // return B
?>
```

好像还是不是很好理解，下面来看这个后静态绑定在MVC框架中的用途，这样更容易理解一些.

## 使用后期静态绑定实现MVC框架中的model基类 ##
在一个MVC框架中，通常使用Model来对数据库抽象，把一张数据库表映射到一个Model的子类。比如，一张person表可以对应一个下面的类:

```
namespace model

class Person extends Model {
	public $name;
	public $age;
	
	public function say($words) {
		...
	}
}
```
那么有一些通用的数据库操作，我们希望可以放到Model类中实现了，这样在各个Model子类中都可以直接调用，比如 `findById()` 用来根据id返回一个Person的实例。这个时候有一个问题就是，Model基类中如何知道各个Model子类的表名呢，我们需要子类把表名告知Model基类，这样才能方便些sql语句。

这时候就可以使用后期静态绑定了，因为后期静态绑定可以在子类调用父类的静态方法时将一些静态域使用自己（子类）的静态域。

```php
class Model {
	public static function findById($id) {
		$tableName = static::tableName();
		$sql = "select * from {$tableName} where id={$id} ";
		// 下面的数据库查询只是一个示例，使用了一个数据库封装
		$row = DBConnector::createSql($sql)->queryOne();
		return new static($row);
	}
}

// 改进子类
class Person extends Model {
	
	public static functin tableName() {
		return 'person';
	}

}
```

这样可以直接使用静态方法获得一个person实例： `Person::findById(1);` 就可以了。
这样其他的model子类也是要在类中实现了静态域 `public static function tableName()` 就能直接使用Model类中findById方法了。

## 扩展 ##
可以在Model中添加更多的方法来使得数据库操作变得简单，比如 deleteByid(), findByArray() 等等，这非常有用。

- 在非静态环境下，所调用的类即为该对象实例所属的类。由于 $this-> 会在同一作用范围内尝试调用私有方法，而 static:: 则可能给出不同结果。另一个区别是 static:: 只能用于静态属性
- 后期静态绑定的解析会一直到取得一个完全解析了的静态调用为止。另一方面，如果静态调用使用 parent:: 或者 self:: 将转发调用信息

```php
<?php
class A {
    public static function foo() {
        static::who();
    }

    public static function who() {
        echo __CLASS__."\n";
    }
}

class B extends A {
    public static function test() {
        A::foo();
        parent::foo();
        self::foo();
    }

    public static function who() {
        echo __CLASS__."\n";
    }
}
class C extends B {
    public static function who() {
        echo __CLASS__."\n";
    }
}

C::test(); 
// 输出:A C C
?>
```

参考： [http://php.net/manual/zh/language.oop5.late-static-bindings.php](http://php.net/manual/zh/language.oop5.late-static-bindings.php "http://php.net/manual/zh/language.oop5.late-static-bindings.php")
