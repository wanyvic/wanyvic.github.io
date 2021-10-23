---
title: Rust笔记(四) 控制流
tags: rust
categories: rust
abbrlink: 825dbcee
date: 2021-10-23 17:12:17
---
# if 表达式
```rust
let number = 3;
if number < 5 {
    println!("less five")
} else if number > 5 && number < 10 {
    println!("more five, less ten")
} else {
    println!("more ten")
}

// 因为是表达式可以赋值
let result = if true {
    3
} else {
    5
};
```
# loop 表达式
```rust
let mut count = 3;
let result = loop {
    if count == 5 {
        break count;
    }
    count += 1;
};
```
# while 循环
while 循环内不能使用`break`关键字
```rust
let mut number = 3;
while number > 0 {
    number -= 1
}
```
# for 循环
```rust
// for in
let arr = [1, 2, 3, 4, 5];
for element in arr.iter() {
    println!("t1 the value is: {}", element);
}

// for range
// (0...arr.len()) 只能正着，反转需要rev()
for i in (0..arr.len()) {
    println!("t2 the value is: {}", arr[i]);
}
//(0..a.len()).rev() 反转
for i in (0..arr.len()).rev() {
    println!("t3 the value is: {}", arr[i]);
}
```