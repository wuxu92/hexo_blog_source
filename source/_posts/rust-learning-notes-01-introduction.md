---
title: "Rust 学习笔记 - 01 介绍"
date: 2018-12-04 09:52:22
categories: Programming
tags:
- Programming
- Rust
---
## 安装

```bash
wget https://sh.rustup.rs -O rustup.sh
bash rustup.rs -y
```
这样会安装一个目录在home目录下，`~/.cargo/{env,bin,registry}`，需要`source ~/.cargo/env` 引入rust需要的环境变量。

## cargo
上面的rustup工具会自动安装rustc， cargo,rustfmt 等工具。

cargo是rust的模块管理，编译管理工具，提供非常多的子命令，可以用来new, build， run 项目，可以用来搜索crate，用来更新模块。

```bash
cargo new --bin project_name
cd project_name
```
cargo new 可以新建项目，使用 `--bin` 或者`--lib` 初始化一个二进制可执行程序项目，或者 lib 制定创建library。 new 出来的项目自动是一个git项目，有一个 `Cargo.toml` 文件是项目描述和依赖声明文件。

## 打印
rusts的一个hello word 程序主体是：
<!-- more -->

```rust
fn main() {
    let mut str = String::new();
    str = String::from("Hello World!");   
    println!("{}", str);
}
```
这三行代码表示：

- rust 使用fn定义函数
- rust程序使用main作为程序入口
- 函数的定义生命和函数体定义和常见编程语言类似
- 代码以行为单位，行尾提供 `;`
- 代码对齐使用4空格，不使用 tab
- `println!()` 有一个特别奇怪的 `!` 表示这是rust中的一个 macro ，不带 ! 的调用表示这是一个函数
- String 是rust提供的内置类型，使用 `::` 操作符调用类的函数，一般 `new()` 函数返回类型的一个实例
- 上面的代码会报告警，因为rust是一种非常严格的语言，告警是因为第一个定义的str没有使用过就被赋值了。
- `println!()` 使用 `{}` 作为占位符


## 模块
编辑 `Cargo.tooml` 文件，在 dependencies 下添加需要的crate， 规则是 `crate_name = "version"` 。crate 可以使用cargo工具搜索： `cargo search rand`。

 然后使用 `crate build` 会下载和编译新添加的crate。

crate 应该是自带文档的，使用 `cargo doc --open` 可以生成这个项目使用的各个crate的文档，并自动在浏览器中打开，非常方便。当然查看rust本身的文档也是很方便的， 使用 `rustup doc` 就可以在浏览器打开rust的本地文档。还可以很方便地搜索。感觉这个文档功能比golang的还要方便一些。

## 类型
虽然大部分时候rust的变量类型是auto inreference 的，但是rust是一个强静态类型的语言，所有的变量有自己的类型，如 String, i32, u32, i64, u64。不同类型之间的比较是有严格限制的。

String 是 A UTF-8 encoded, growable string 类型， str 是 String slices。 这两者是有区别的。String 和 str 各自有自己的方法。

```rust
let str = "abc";
let s = String::new(); // or s = String.from(str);
```

类型转换， shadow variable.
```rust
let guess_num = String::from("123");
// 1 通过 ::<i32>() 指定类型
let guess_num = guess_num.trim().parse::<i32>()
    .expect("parse to number failed.");
// 指定 guess_num 的新类型
let guess_num : i32 = guess_num.trim().parse()
    .expect("parse to number failed.");
```

## match
matchs 是rust中一个比较特殊的概念，在其他语言中比较少见，有点类似 `switch` 的,不过在Rust中可以做更多的事情。

下面是一个稍微改造的rust官方的教学程序：


```rust guessing_number.rs
extern crate rand;

use std::io;
use rand::Rng;
use std::cmp::Ordering;

fn main() {
    println!("Guess the number!");
    let rand_num = rand::thread_rng().gen_range(1, 100);
    println!("Get a random number: {}", rand_num);
    loop {
        let mut guess_num = String::new();
        print!("Please input a number:");
        io::stdin().read_line(&mut guess_num)
            .expect("Failed to read line");
        println!("You Input Guess Number: {}", guess_num.trim());
        // let guess_num = guess_num.trim().parse::<i32>();
        // if guess_num.is_err() {
        //     println!("input ERROR!, skip");
        //     continue;
        // }
        //let guess_num = guess_num.unwrap();
        // or
        let guess_num = match guess_num.trim().parse::<i32>() {
            Ok(num) => num,
            Err(_) => {
                println!("invalid number.");
                continue;
            },
        };
        match guess_num.cmp(&rand_num) {
            Ordering::Less => println!("Less"),
            Ordering::Greater => println!("Greater"),
            Ordering::Equal => {
                println!("Equal!!");
                break;
            },
        }
    }   
} 
```
