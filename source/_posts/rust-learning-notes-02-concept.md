---
title: "Rust 学习笔记 02 通用编程概念"
date: 2018-12-04 09:55:54
categories: Programming
tags:
- Programming
- Rust
---

## 数据类型
默认rust的变量都是 immutable 的，不能对其重新赋值。但是由于rust可以 shwdow variable，所以可以重新生命同名的变量实现一种可以理解为显式的动态变量的效果。

```
let str = String::from("123");
str = str.trim().exepct::<i32>().expect("");  // Error, str is immutable
let str = 123; // OK, shadowing old str variable, now is an i32 variable
```

rust 提供四种标量类型（scalar types) ： Integerts, Floating-point numbers, booleans, characters。 具体数据类型包括：`i8, u8, i16, u16, i32, u32, i64, u64, isize, usize, f32, f64, true, false`。默认int类型为 i32, 浮点数为 f64。

rust的计数方法也比较独特，可以添加有类型后缀，数字中间可以有下划线作助记符（visual separator): `98_222, 0xfff, 0o77, 0b11011_0011, b'A'` 这样。

<!-- more -->
Rust 的char类型不同于 c或cpp的char，它是 unicode Scalar Value，这表示一个char类型不只是表达ASCII了，还可以用来表示一个汉字或和文字符（CJK）等表意文字,甚至可以表示重音字母(accented letters), emoji.

## 组合类型
Rust提供两种基础的组合类型： tuples 和Arrays。

Rust 的tuple概念可以通过下面的示例明白：

```rust
let tup = (100, 20.1, true);
// or: let tup : [ i32,f64, bool] = ...;
println!("tuple destructuing: {} {} {}.", tup.0, tup.1, tup.2);
// destructuing
let (one, two, three) = tup;
println!("get variables: {} {} {}", one, two, three);
```
稍微注意的是，tuple的按下标索引使用 `.0` 这样的方式。 Rust 会在编译期就检查tuple的各个元素的数据类型，以及保证 destructuring 时元素的个数不会越界。Tuples的使用是非常安全的。

Tuple 的各个元素可以有不同的数据类型，而Array的主要特点是所有元素必须有相同的数据类型。Rust的数组也是固定长度的，初始化之后就不能调整大小，同样的，数据类型和数组大小的检查会在编译期进行.

Rust 提供运行时数组越界检查，试图访问数组定义之外的元素会导致panic（而不是像C 或 CPP那样返回一个不可预测的东西），Array的示例如下：

```rust
let arr = [100, 200, 300];
println!("get arr {}", arr[0]);
let mut idx = String::new();
std::io::stdin().read_line(&mut idx).expect("read from input failed.");
let idx : usize = idx.trim().parse().expect("as int");
println!("get item for index {} as: {}.", idx, arr[idx]);
```

## 函数
像之前提到的，Rust 使用 `fn` 关键字生命函数。在Rust中，通用的函数和变量命名使用 snake case. 就是全部使用小写，单词与单词之间使用下划线分隔。

Rust中对函数调用和定义的先后顺序没有要求，只要在某个地方定义了就可以了，这和其他语言不太一样。

函数参数的声明和golang有点类似，需要提供参数的类型，如 `fn func(x: i32) {}`。

Rust中statement 和 expression 是有明确区分的。statement 是一些指令执行一些操作，但是没有返回值的。比如 `let x = 6;` 是一个 statement，本身是没有return value 的，所以像 `let x = (let y = 10);` 这样的声明在Rust中是错误的；这和类C的语言也很不一样。 expression 是用来计算某些东西，可以是statement的一部分，比如 `5+6` 是一个expression，计算的结果的是数值 11 。调用一个函数，调用宏， `{}` 都是expression。expression的结尾没有 `;`，如果加上 `;` expression就成了 statement，就没有返回值了。下面是一个示例：

```rust
fn main() {
    let x = 5;
    let y = {
        let x = 3;
        x * 3   // NO ; here
    };
    println!("x = {}; y = {}.", x, y);  // x = 5, y = 6
}
```

函数的返回值生命格式如下： `fn double(x: i32) -> i32 { x * 2 };` 这样的函数生命看起来很清爽简洁。

## 注释
Rust的注释格式和C风格一样的行注释和段注释。推荐使用行注释。段注释是工具不友好的（git, diff, grep 等），不推荐使用。

## 控制流
`if expression`，分支判断语句，和golang类似，if 的 condition 不需要小括号包裹。Rust中 if 表达式有时候被称作 *arms*。一个极简的示例：

```rust
let i: i32 = 100;

if double(i) == 200 {
    println!("match.");
} else { 
    println!("missed");
} 
```
Rust中if的条件必须是bool类型，而不是类C的自动转换操作。 如 `let x = 3; if  x { ... };` 这样的语句是不能编译通过的。

虽然 `if .. else if` 语句可以有很多分支，但是有超过一个 else if 的时候就应该考虑重构代码了，比如使用 `match`。

rust 允许在 let statement 中使用 If expression。比如下面的代码，理解 statement和expression对理解这样的代码很关键。

```rust
let x = 10;
let y = if double(x) == 20 { ...; 20 } else { 0 };
```
记住快代码得到的结果的最后一个expression的值， statement没有值语义。在上面的代码中if两个分支返回的的值类型必须一样，否则 y 的类型在编译期是不确定的，这在Rust中是不允许的。

Rust 提供 `loop, while, for` 三个常见的实现循环的语法。各自的用法也比较直观：

```rust
loop { ... }
while condition_expression { ... }
for ele in iterable { ... }
```
for 循环遍历的是一个 Iterable的对象，比如遍历一个数组，需要使用 `for item in arr.iter() { ... }` 这样的方式。

Rust library 提供一种Range的类型，用于生成一个前闭后开区间序列，常用在 for 循环中，比如： 

```rust
for idx in 0..6 { println!("range to {}.", idx); } 
for idx in (0..6).rev() { println!("range to {}.", idx); }
```
一个简单的获取第n个fibonacci数的算法：

```rust
fn fib(i : usize) -> u64 { 
    let mut s = 1u64;
    let mut e = 1u64;
    for _  in 2..i { 
        let t = e; e = s+e; s = t;
    } e     // e 放到这里也是可以的
}
```
这段代码即使是 `fib(1)` 调用也是没问题的，` 2..i` 并不会报错。
