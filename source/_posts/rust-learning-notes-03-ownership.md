---
title: "Rust 学习笔记 03 Understanding Ownership"
date: 2018-12-04 09:57:24
categories: Programming
tags:
- Programming
- Rust
---

Ownership 是rust特有的概念，使得rust在不需要gc的情况下能够保证内存安全。这一章讨论Rust中的Ownership, 还有一些相关的特性： borrowing, slice, 以及Rust数据在内存中的布局。

## Ownership
Rust中的ownership的规，为了准确这里记录原文：

1. Each value in Rust ha a variable that is called its owner.
1. There can only be one owner at a a time.
1. when the owner goes out of scope, the value will be dropped.

Rust提供与其他语言（C）类似的变量作用于概念，基本就是块级作用域。

之前提到的4中常量类型变量比较简单，都是存储在栈上面，他们的作用域也非常简单，现在要考虑更复杂的结构，以及他们如何存储在堆上，以及Rust如何知道什么时候该清理堆上的数据。
<!-- more -->

Rust提供的内存管理模式和CPP中的RAII(Resource Acquisition Is Initialization) 模式相似，就是当变量离开作用域的时候自动释放资源。但是Rust原生的这一特性会更加复杂:在复杂的赋值情况下如何处理。

**Rust 中变量与数据的交互： Move**:
在其它语言中像下面一段简单的代码会报编译错误：

```rust
let s1 = String::from("owner");
let s2 = s1;
println!("s1 is {}.", s1);
```
编译这段代码，报错如下：

```bash
 --> src/main.rs:4:27
  |
3 |     let s2 = s1;
  |         -- value moved here
4 |     println!("s1 is {}.", s1);
  |                           ^^ value used here after move              
  = note: move occurs because `s1` has type `std::string::String`, which does not implement the `Copy` trait

error: aborting due to previous error

For more information about this error, try `rustc --explain E0382`.
error: Could not compile `ownership`.

To learn more, run the command again with --verbose.
```
具体的原因其实就是上面提到的三条规则中的第二条： a value cannot be owned by more than one variable. 

Rust中的赋值语义默认就是 `Move` 语义的，不像其他语言中的copy语义。 `s2 = s1;` 表示将 s1 变量 move 到 s2；这时只有 s2 是一个合法的变量， s1 不能再使用。

由于Rust默认不会做 deep copy， 所以任何的赋值操作都可以认为是很轻量，运行时基本无开销的。

**Clone**: 上面提到的Rust默认的赋值操作是move语义的，如果我们就是需要变量的一个deep copy 的副本呢？这时候我们应该使用 `clone` 方法（`let s2 = s1.clone()`），使用 clone 方法的思路在其他语言，如Java 中也常遇到。显然使用 clone 方法比move语义的开销大。

像int这样的 Stack-Only 的数据类型当然总是执行Copy语义的，并没有Move语义，因为那样并不能带来什么好处。Rust有一个特殊的称为 `Copy` 的标记，如果一个类型拥有 Copy trait, 发生赋值时，旧变量仍然是可用的。但是如果类型，或者类型中任何元素实现的 `Drop` trait，那么将不能标记为Copy。

常见的带有Copy语义的类型：

- 所有integer类型
- bool类型，值为 true or false
- 所有浮点数类型， f32, f64
- Tuples，但是Tuple的元素本身也必须是Copy的。

将变量传递为函数作为参数类似与赋值语义，会发生 Move 或者 Copy 动作。如果是Move的，那么函数调用的效果会和我们熟悉的其他语言截然不同： 会使得传递给函数调用作为参数的变量在函数调用之后失效，也就是无法再使用：因为变量的ownership被传递给函数参数了，在函数结束时实参作用域结束会自动调用Drop释放资源；当然Copy类型不会发生这种情况。下面是一段示例：

```rust
fn print(s: String) { println!("{}", s);}

fn test_func() {
    let s = String::from("123");
    print(s);
    let s2 = s; // Error: s is no longer valide
}
```

官方提供了一段代码可以很好说明ownship在函数调用过程中的变换：

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope
    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here
    let x = 5;                      // x comes into scope
    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it’s okay to still
                                    // use x afterward
} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.

```

如果熟悉C风格的函数返回值，可以猜到Rust中的返回值也会导致Ownership的转移。

Rust中变量的Ownership 遵循同一个规则： 把变量赋值给其另一个变量会转移（Move）它。一个在堆上面拥有数据的变量出了作用域（out of scopr），这个值会被drop清理，除非它被move为另一个变量所有。

这样如果我们希望传递给函数变量还需要继续使用，那么需要作为参数的变量需要再传递出来，这样有点过于麻烦了。Rust提供了名为 *reference* 的机制来解决这个问题。

## Reference and Borrowing
reference 就可以理解为C 中的引用，在函数定义和调用两个地方都使用 `&` 操作符标识即可。Rust中没有指针，所以在函数调用处使用 `&` 并不代表取地址，而是传递引用：

```rust
fn print(s : &String) { println!("{}", s);} // 函数的声明需要使用 & 指示接受的参数是一个引用
let s = String.from("abc");
print(&s);  // 创建一个到s的引用
let s2 = s;
```
`&s` 创建一个到s的引用但是不拥有s(not own it)，由于不拥有s，所以引用退出作用域的时候不会drops其值。

引用的机制也说明了前面提到的三条规则之一：一个值只有一个变量持有（owned by)，因为引用不持有变量。

Rust中将引用作为函数参数的用法称作 *borrowing*。‘

通过引用传递可以不改变变量的ownership，但是不能在函数内改变遍历， 比如 `fn print(s: &String) { s.push_str(" - 123"); } ` 是错误的，如果需要再函数内部改变变量的值，需要使用显式的 mutable reference 函数声明和函数调用，注意，需要函数声明和函数调用同时支持mut, 并且变量本身就是声明为mutable的。

```rust
fn print2(s: &mut String) { s.push_str(" add"); print!("{}", s); }
let mut s = String::from("123");
print2(&mut s);
```
mutable reference 和C 语言的引用作用比较像了，其限制也类似C语言，变量在作用域内只能有一个可变引用。这一个限制是的变量的可变性得到极大的限制，其直接的好处是在编译期可以阻止数据争用出现( prevent data races at compile time)。

Rust的引用机制解决了C/CPP中引用的限制，C 中引用必须在声明时进行初始化，而且不能改变引用指向其他的变量。而Rust中的普通引用都是不能修改对象的，可以随意修改指向的对象，只有两个限制：

- 可变引用在一个作用域内只能有一个
- 一个变量在作用域内，mutable引用和 immutable 引用不能同时存在

**Dangling References**， 类似于悬挂指针，Rust的引用似乎也有类似的悬挂引用问题。但是Rust的编译器会在编译期保证引用不会超出其所引用的变量的作用域。也就是Rust中引用的作用域不能超过其所指向的变量的作用域。这在一开始可能有些令人疑惑，比如下面的代码：

```rust
let mut s = String::from("123");

let mut r1 = &s;
let s2 = String::from("abcd");  // Error: `s2` does not live long enough:
                            //  borrowed value does not live long enough
r1 = &s2;
```
类似的代码引出了Rust中另一个特有的概念： *lifetime*，这要到后面再学习。

关于悬挂指针的示例，官方有两段代码，第一段时有问题的，后面不是用引用的是没问题的：

```rust
fn dangle() -> &String { // dangle returns a reference to a String

    let s = String::from("hello"); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!

fn no_dangle() -> String {
    let s = String::from("hello");
    s      // Ownership is moved out, and nothing is deallocated
}
```

所以，关于引用官方提供了下面两条原则：

- At any given time, you can have either (but not both of) one mutable reference or any number of immutable references.
- References must always be valid.

## Slice
在很多语言里面 Slice 都是很常用，很方便的工具。Rust中的Slice有很多独特的性质。

Slice 是一种没有所有权的数据类型。slice允许访问集合中一段连续的元素而不是整个集合，这就确立了 slice 作为引用本质。

获取slice的的方法，

```rust
let s = String::from("Hello world!");
let sli = &s[0..5];  // &s[..3], &s[2..], &s[..] 都是可以的
```
slice总是引用类型的，并且字符串字面量也总是slice，类型为 `str`，所以一个返回字符slice的函数可以定义为： `fn first_word(s : &String) -> &str { ... }`。 而其他类型的slice 的类型类似为 `&[u16], &[i32], &[f32]` 等。

```rust
let s = "Hello world!";     // s here type is &str

// 下面的函数定义更常见，可以使用 &str 作为实参或者很方便地使用 &String [..]
fn first_world(s : &str) -> &str { ... }
```
