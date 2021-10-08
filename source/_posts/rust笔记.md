---
title: rust笔记
abbrlink: 415a55af
date: 2021-09-28 18:36:12
tags:
---
## rust 笔记
### 安装
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
#### 更新
```
rustup update
```
#### 卸载
```
rustup self uninstall
```
#### 编译
```
rustc xxx.rs
```
#### 命名规范
snake_case: 文件名
CamelCase:

## Cargo
cargo 是rust的构建系统和包管理工具
### 创建项目
```
cargo new xxx
```
### 编译 
```
cargo build
cargo build --release
```
### 运行
```
cargo run
```
### 检查
```
cargo check # 检查代码，cargo check比cargo build运行时间快的多
```