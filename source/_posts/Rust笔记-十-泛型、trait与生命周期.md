---
title: Rust笔记(十) 泛型、trait与生命周期
tags: [rust, 泛型, trait, 生命周期]
categories: rust
abbrlink: 7c516f81
date: 2021-10-23 18:51:48
---
# 泛型
## 泛型函数
```rust
// 只展示语法形式，add函数编译会报错
fn add<T>(a: T, b: T) -> T
{
    a + b
}
```
## 结构体泛型
```rust
struct Point<T, U> {
    a: T,
    b: T,
    c: U,
}
impl<T, U> Point<T, U> {
    fn new(a: T, b: T, c: U) -> Point<T, U> {
        Point { a, b, c }
    }
}
// 泛型偏特化 只有在具体类型才有new2方法
impl Point<i32, i64> {
    fn new2(a: i32, b: i32, c: i64) -> Point<i32, i64> {
        Point { a, b, c }
    }
}
```
## 枚举泛型
```rust
enum Some<T> {
    A(T),
    None,
}
```
# trait
## 定义
```rust
pub trait A {
    fn string(&self) -> String {
        String::from("impl")
    }
}
```
## 实现
```rust
// 无法为外部类型来实现外部的trait
struct ImplA {}
struct ImplB {}
impl A for ImplA {}
impl A for ImplB {
    //重写
    fn string(&self) -> String {
        String::from("implB")
    }
}
```
## trait 约束
```rust
// Trait 约束
fn s0(a: impl A, b: impl A) {}
// 要求参数同时实现A和Display的trait
fn s1(a: impl A + std::fmt::Display) {}
// trait bound（trait 与泛型绑定）
fn s2<T: A + std::fmt::Display>(a: T, b: T) {}
// where 子句 trait 约束
fn s3<T, U>(a: T, b: U)
where
    T: A + std::fmt::Display,
    U: A + std::fmt::Display,
{
    // sss
}
```
## 返回值为 trait
```rust
// 返回只能为同一种类型 trait
fn s4(flag: bool) -> impl A {
    // if flag {
    ImplB {}
    // } else {
    // ImplA {}    // error
    // }
}
```
# 泛型 + trait
```rust
// where 子句对T进行trait约束
// 两个相同类型相加，要求T实现了`std::ops::Add`的trait
fn add<T>(a: T, b: T) -> T
where
    T: std::ops::Add + std::ops::Add<Output = T>,
{
    a + b
}

// 两个不同的类型相加，前提可转换，即实现了std::convert::From<U>或std::convert::Into<T>的trait
// 例如: add2(1, 2.0)
fn add2<T, U>(a: T, b: U) -> T
where
    T: std::ops::Add + std::ops::Add<Output = T> + std::convert::From<U>,
{
    a + T::from(b)
}

fn add3<T, U>(a: T, b: U) -> T
where
    T: std::ops::Add + std::ops::Add<Output = T>,
    U: std::convert::Into<T>,
{
    a + U::into(b)
}
```