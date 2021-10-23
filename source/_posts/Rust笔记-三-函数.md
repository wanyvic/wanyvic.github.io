---
title: Rust笔记(三) 函数
tags: [rust, function, 函数]
categories: rust
abbrlink: a0f35e48
date: 2021-10-23 16:58:03
---
# 函数
函数名使用蛇形命名法`snake case`
```rust
fn main() {
    let result = add_four(7);
    println!("result:{}", result);
}

// parameter 形参 argument 实参
// add_four 蛇形命名法
fn add_four(x: i32) -> i32 {
    let y = {
        let x = x + 1;
        x + 3
    };
    y // return y; 可以使用return y; 也可以直接使用表达式返回
}
```
# 表达式与语句区别
- 语句（statement）执行操作但不返回值的命令。
- 表达式（expression）会进行计算并返回一个结果的指令。
```rust
// 语句:
    let x = 5;
// 表达式:
    {
        let x = 1;
        x + 3    // 无分号。返回最后一个表达式的结果
    }
```