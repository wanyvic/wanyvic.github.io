---
title: Rust笔记(一) 安装
tags: rust
categories: rust
abbrlink: 6af9b2cc
date: 2021-10-23 15:55:00
---
# 安装
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
## 更新
```bash
rustup update
```
## 卸载
```bash
rustup self uninstall
```
## 编译
```bash
rustc xxx.rs
```
## 命名规范
`snake_case`: 文件名  
`CamelCase`:

# Cargo
cargo 是rust的构建系统和包管理工具。
## 创建项目
```bash
cargo new xxx # 创建二进制项目
cargo new xxx --lib # 创建库项目
```
## 安装依赖
```bash
cargo install xxxx
```
## 编译 
```bash
cargo build
cargo build --release
```
## 运行
```bash
cargo run
```
## tests
```bash
cargo test
```
## 检查
检查代码，`cargo check`比`cargo build`运行时间快的多
```bash
cargo check
```

### panic 中断
`Cargo.toml`
```toml
[profile.release]
panic = 'abort'
```

## 使用外部package
在 Cargo.toml的
`[dependencies]` 添加package

## cargo 换源
`.cargo/config`
```toml
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"

replace-with = 'tuna'
[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"

[net]
git-fetch-with-cli = true
```