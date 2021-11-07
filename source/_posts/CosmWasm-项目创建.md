---
title: CosmWasm 项目创建
tags:
  - wasm
  - cosmos
  - 区块链
categories:
  - 区块链
abbrlink: b2bb0ade
date: 2021-11-05 17:09:52
updated: 2021-11-05 17:09:52
copyright:
---
# 使用[cw-template](https://github.com/CosmWasm/cw-template)模板初始化

## 环境要求
- Rust 1.55+

## 安装cargo-generate
```bash
cargo install cargo-generate --features vendored-openssl
cargo install cargo-run-script
```
## 创建新的合约
```bash
cargo generate --git https://github.com/CosmWasm/cw-template.git --name PROJECT_NAME
```
使用旧版本需要加上版本号作为参数
```bash
cargo generate --git https://github.com/CosmWasm/cw-template.git --branch <version> --name PROJECT_NAME
```
Example:
```bash
cargo generate --git https://github.com/CosmWasm/cw-template.git --branch 0.16 --name PROJECT_NAME
```