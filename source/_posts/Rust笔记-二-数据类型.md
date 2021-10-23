---
title: Rust笔记(二) 数据类型
tags: rust
categories: rust
abbrlink: 1637e2fc
date: 2021-10-23 16:55:00
---
# 变量的可变性
可变变量需要加上`mut`关键字
```rust
let a = 1;
a = 2; // cannot assign twice to immutable variable `a`

let mut b = 1;
b = 2; // ok!
```

# 常量
```rust
const MAX: i32 = 100;
```
# 隐藏
隐藏（shadow）
```rust
let x = 5;
let x = x + 1;
```

# 标量类型(scalar type)
```rust
let u1: u8 = b'A';
let u2: u16 = 0xff;
let u3: u32 = 0o77;
let u4: u64 = 232_212;
let u5: u128 = 232_212;

let i1: i8 = 127;
let i2: i16 = 254;
let i3: i32 = 254;
let i4: i64 = 254;
let i5: i128 = 254;

let f1: f32 = 254.0;
let f2: f64 = 254.0;

let b: bool = false;

let c1: char = '我';
let c2: char = '😂';

// 默认值为i32和f64
let i = 254; // let i: i32 = 254;
let f = 254.0; // let f: f64 = 254.0;
```
debug模式下会检测溢出，代码触发panic，release模式下不会检测溢出，会执行二进制补码“环绕”。

# 复合类型(compound type)
## 元组(tuple)
```rust
let t: (u8, f64, bool) = (240, 3.2, false);
let t2 = (240, 3.2, false);
println!("{},{},{}", t.0, t.1, t.2);
```
## 数组(array)
```rust
let arr = [7; 5]; // [elem, len] 表示: 5个7
let arr1: [i32; 5] = [1, 2, 3, 4, 5];
let arr2 = [3, 3, 3, 3];
println!("{},{}", arr.len(), arr2.len());

println!("{}", arr2[arr[0]]);// 访问越界 thread 'main' panicked at 'index out of bounds: the len is 4 but the index is 7', src/main.rs:21:20
```