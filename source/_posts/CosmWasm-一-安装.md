---
title: CosmWasm 一 安装
abbrlink: e627dad1
date: 2021-11-02 16:22:39
tags: [wasm, cosmos, 区块链]
categories: [区块链]
copyright:
---
# rust安装省略
# 升级rust并安装wasm32 target
rust版本要求1.51.0+
```shell
rustup default stable
cargo version
# If this is lower than 1.51.0+, update
rustup update stable

rustup target list --installed
rustup target add wasm32-unknown-unknown
```
# 安装wasmd
```
git clone https://github.com/CosmWasm/wasmd.git
cd wasmd
# replace the v0.20.0 with the most stable version on https://github.com/CosmWasm/wasmd/releases
git checkout v0.20.0
make install

# verify the installation
wasmd version
```
# 建立环境
## pebblenet 测试网
Block Explorer: https://block-explorer.pebblenet.cosmwasm.com

### 设置pebblenet的参数到本地临时环境变量
```bash
source <(curl -sSL https://raw.githubusercontent.com/CosmWasm/testnets/master/pebblenet-1/defaults.env)
```

### 添加wallet
```
# add wallets for testing
wasmd keys add wallet
# output
>
{
  "name": "wallet",
  "type": "local",
  "address": "wasm13nt9rxj7v2ly096hm8qsyfjzg5pr7vn5saqd50",
  "pubkey": "wasmpub1addwnpepqf4n9afaefugnfztg7udk50duwr4n8p7pwcjlm9tuumtlux5vud6qvfgp9g",
  "mnemonic": "hobby bunker rotate piano satoshi planet network verify else market spring toward pledge turkey tip slim word jaguar congress thumb flag project chalk inspire"
}

wasmd keys add wallet2
```
**注**：这里可能会报错，此命令需要gui密码库支持。如果是无gui的terminal用以下命令代替：
```
wasmd keys add wallet --keyring-backend file
```