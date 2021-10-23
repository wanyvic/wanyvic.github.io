---
title: Rust笔记(六) 结构体
tags: [rust, struct, 结构体]
categories: rust
abbrlink: ba2042f3
date: 2021-10-23 17:51:19
---
# 定义结构体
```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
// 没有结构体继承
struct User2 {
    a: String,
    ..User,
} // error

// unit-like struct 空结构体
struct Empty;
```
# 初始化结构体变量
```rust
let user1 = User {
    username: String::from("aaa"),
    email: String::from("abc@abc.com"),
    sign_in_count: 1,
    active: true,
};
// 同名简写
let username = String::from("bbb");
let user2 = User {
    username,
    email: String::from("abc@abc.com"),
    sign_in_count: 1,
    active: true,
};
// 更新简写
let user3 = User {
    username: String::from("ccc"),
    ..user2
};
```
# 定义方法
```rust
struct Rectangle {
    width: u32,
    length: u32,
}
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.length
    }
    // 关联函数 associate function(静态方法) 第一个参数不是self
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            length: size,
        }
    }
}
```
# tuple struct
想给类型相同的tuple做区分，但是里面的元素没名
```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);
```
# 结构体打印调试信息
```rust
// struct 上面加 #[derive(Debug)]
#[derive(Debug)]
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
let user = User {
    username: String::from("sss"),
    email: String::from("abc@abc.com"),
    sign_in_count: 1,
    active: true,
};
println!("{:?}", user);
println!("{:#?}", user);    // 带换行
```
# struct 数据所有权
- struct 拥有其所有数据所有权
-  如果 struct 存引用，则需要使用生命周期。（生命周期保证struct实例有效，里面的引用就有效）