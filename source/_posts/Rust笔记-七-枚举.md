---
title: Rust笔记(七) 枚举
tags: [rust, enum, 枚举]
categories: rust
abbrlink: 274a678a
date: 2021-10-23 18:07:34
---
# 枚举定义和使用
```rust
enum IpAddrKind {
    V4,
    V6,
}
let four = IpAddrKind::V4; // 枚举值
let six = IpAddrKind::V6; // 枚举值

// enum 附带数据
enum IpAddr {
    V4(String),
    V6(String),
}
let four = IpAddrKind::V4(String::from("127.0.0.1"));
let six = IpAddrKind::V6(String::from("::1"));

// 实现枚举方法（与struct 类似）
impl IpAddr {
    // ...
}
```
# Opthion枚举
```rust
// 预导入库中定义了结构
enum Option<T> {
    Some(T),
    None,
}
```
# 模式匹配
- refutable 可反驳模式
- irrefutable 不可反驳模式
## match 匹配
```rust
let some_number = Some(5);
let some_string = Some("string");
let undefined_number: Option<i32> = None;
// 必须穷举所有可能
match some_number {
    Some(s) => {
        println!("{}", s + 1);
        Some(s + 1)
    }
    None => None,
};
// - => () 通配符
match some_number {
    Some(3) | Some(5) => println!("3 or 5"),
    Some(x) => println!("{}", x),
    _ => println!("None"),
};
```
## if let 匹配
if let [类型] = [变量] { }
```rust
let some_number = Some(5);
if let Some(3) = some_number {
    println!("{}", 3);
} else if let Some(x) = some_number {
    println!("{}", x);
} else {
    println!("None");
}
```