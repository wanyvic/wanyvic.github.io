---
title: Rust笔记(八) 通用集合
tags: rust
categories: rust
abbrlink: ab025fe7
date: 2021-10-23 18:26:27
---
# Vector
```rust
let v: Vec<i32> = Vec::new();
// 用vec! 宏带值初始化
let mut v1 = vec![1, 2, 3, 4];
// 添加
v1.push(5);

// 读取
let c = &v1[1]; // 下标越界即panic

// get方法返回Option<T>
match v1.get(1) {
    Some(x) => println!("{}", x),
    None => println!("None"),
}

// enum + vector 存储多个类型的值
enum Cell {
    Int(i32),
    Float(f64),
    Text(String),
}
let row = vec![
    Cell::Int(3),
    Cell::Float(2.0),
    Cell::Text(String::from("str")),
];
```
# String
```rust
let mut s = String::new();
let s1 = String::from("ssss");
let s2 = "ssss".to_string();
let s3 = &s1[..];
// push_str
s.push_str(&s3);
s.push('l');
let s4 = s + &s1; // 使用s的所有权
// println!("{}", s); //error

// format
println!("{}", format!("{}-{}-{}", s1, s2, s4));
// string index
for i in s4.bytes() {
    println!("{}", i);
}
for i in s4.chars() {
    println!("{}", i);
}
```
# HashMap
```rust
// 引入Package
use std::collections::HashMap;
let mut table: HashMap<i32, f32> = HashMap::new();

// insert
table.insert(23, 3.0);

// collect
let key = vec![10, 20];
let value = vec![1.0, 2.0];
let mut table2: HashMap<_, _> = key.iter().zip(value.iter()).collect();

// 所有权
// 实现了Copy 就赋值，没实现则移交所有权

// 访问
if let Some(x) = table2.get(&10) {
    println!("v:{}", x);
} else {
    println!("none");
}
// 遍历
for (k, v) in &table2 {
    println!("{}:{}", k, v);
}
//  not exist insert
let a = table2.entry(&10).or_insert(&3.0);
println!("{:?}", table2.entry(&10)); // OccupiedEntry
println!("{:?}", table2.entry(&30)); // VacantEntry
```