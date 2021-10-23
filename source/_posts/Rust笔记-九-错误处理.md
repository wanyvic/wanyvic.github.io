---
title: Rust笔记(九) 错误处理
tags: [rust, 错误处理]
categories: rust
abbrlink: b9e41b5c
date: 2021-10-23 18:41:09
---
# 不可恢复错误: panic
```rust
panic!("str");
```
# 可恢复错误: Result<T,E>
```rust
// 定义在标准库
// enum Result<T,E> {
//     Ok(T),
//     Err(E),
// }
let f = match std::fs::File::open("a.txt") {
    Ok(file) => file,
    Err(error) => match error.kind() {
        std::io::ErrorKind::NotFound => match std::fs::File::create("a.txt") {
            Ok(file) => file,
            Err(error) => panic!("{:?}", error),
        },
        s => panic!("{:?}", s),
    },
};
// 闭包简写
let f = std::fs::File::open("a.txt").unwrap_or_else(|error| {
    if error.kind() == std::io::ErrorKind::NotFound {
        std::fs::File::create("a.txt").unwrap_or_else(|error| panic!("{:?}", error))
    } else {
        panic!("{:?}", error)
    }
});

// unwrap()
let f = std::fs::File::open("a.txt").unwrap();
// 如果能打开则返回Ok(x)内的值,如果出错则调用panic

// expect() 与unwrap() 一样但可以附加错误信息
let f = std::fs::File::open("a.txt").expect("无法打开");
```
# 传播错误
返回值使用`Result<T,E>`来传播错误
```rust
// ? 只能用返回值类型Result的函数
// ? 如果执行失败则return Error
use std::io::Read;
fn read_string_from_file() -> Result<String, std::io::Error> {
    let mut s = String::new();
    std::fs::File::open("a.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```