---
title: Rust笔记(五) 所有权
tags: [rust, 所有权]
categories: rust
abbrlink: b1e3a0f
date: 2021-10-23 17:31:13
---
# 所有权规则
- Rust中每一个值都有一个对应的变量作为它的所有者
- 在同一时间内，值有且仅有一个所有者
- 当所有者离开自己的作用域时，它持有的值就会被释放掉
```rust
let mut s = String::from("hello");
println!("{}", s);
s.push_str(" world");
println!("{}", s);

let s2 = s; // 所有权转移
println!("{}",s); //error: borrow of moved value: `s`

let s2 = s.clone(); // clone()对stack和Heap数据进行深拷贝
println!("{}", s); // ok
```
# Copy trait vs Drop trait
1. 实现了 Copy trait 旧变量赋值后仍然可以使用
2. 实现了Drop trait 就不允许实现Copy trait
3. 简单标量类型都实现了Copy
4. 需要Heap分配的类型都没实现Copy
5. tuple 内所有字段都是实现Copy就可以Copy
# 引用和借用
- `&`与c++类似，引用某值不获取所有权  
- 将引用作为函数参数的行为称之为借用
## 数据竞争三条规则
- 两个或多个指针同时访问同一个数据
- 至少有一个指针用于写入数据
- 没有使用任何机制来同步对数据的访问

如果都满足，就有可能出现数据竞争、报错。

Rust的引用规则：
- 一个可变的引用
- 任意数量不可变引用
只能满足其一
## 悬垂引用 dangling refrences
- 类似与cpp的野指针
- rust内 编译器可保证引用永远都不是悬垂引用
# 切片
```rust
let s = String::from("hello world");
println!("{}", &s[0..5]);
println!("{}", &s[6..]);
println!("{}", &s[..]);
```