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
```bash
rustup default stable
cargo version
# If this is lower than 1.51.0+, update
rustup update stable

rustup target list --installed
rustup target add wasm32-unknown-unknown # 安装wasm32的target
```
# 安装wasmd
```bash
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
注意备份助记词
```bash
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
**Error: No such interface “org.freedesktop.DBus.Properties” on object at path /**  
**注**：如果你使用的是`headless mode`的linux这里可能会报错。可以使用以下命令代替：

```bash
wasmd keys add wallet --keyring-backend file
# 此命令会在--keyring-dir中写入文件
```
### 查看wallet
```bash
wasmd keys list #or
wasmd keys list --keyring-backend file
```

### 水龙头申请代币
```bash
JSON=$(jq -n --arg addr $(wasmd keys show -a wallet) '{"denom":"upebble","address":$addr}') && curl -X POST --header "Content-Type: application/json" --data "$JSON" https://faucet.pebblenet.cosmwasm.com/credit
# 如果使用file的keyring-backend，则使用下列命令
JSON=$(jq -n --arg addr $(wasmd keys show -a wallet --keyring-backend file) '{"denom":"upebble","address":$addr}') && curl -X POST --header "Content-Type: application/json" --data "$JSON" https://faucet.pebblenet.cosmwasm.com/credit
```
### 查询余额
区块浏览器查询钱包地址: <https://block-explorer.pebblenet.cosmwasm.com/>

### 有用的环境变量
```bash
# bash
export NODE="--node $RPC"
export TXFLAG="${NODE} --chain-id ${CHAIN_ID} --gas-prices 0.001upebble --gas auto --gas-adjustment 1.3"

# 如果你用的是zsh则添加以下环境变量
# zsh
export NODE=(--node $RPC)
export TXFLAG=($NODE --chain-id $CHAIN_ID --gas-prices 0.001upebble --gas auto --gas-adjustment 1.3)
```
# 下载合约示例代码和编译
```bash
git clone https://github.com/InterWasm/cw-contracts
cd cw-contracts
git fetch --tags
git checkout nameservice-0.11.0
cd contracts/nameservice

cargo wasm # 此处若依赖下载报错,Cargo则需要换源
RUSTFLAGS='-C link-arg=-s' cargo wasm #以此命令生成最下wasm
```

# 上传和交互

## 上传
将代码上传到链上
```bash
RES=$(wasmd tx wasm store target/wasm32-unknown-unknown/release/cw_nameservice.wasm --from wallet $TXFLAG -y --keyring-backend file)
CODE_ID=$(echo $RES | jq -r '.logs[0].events[-1].attributes[0].value')

# 查找当前codeId生成的contract，如果不存在则返回空
wasmd query wasm list-contract-by-code $CODE_ID $NODE --output json

# 你也可以从链上下载这个wasm并且和本地检查一下是否一致
# you can also download the wasm from the chain and check that the diff between them is empty
wasmd query wasm code $CODE_ID $NODE download.wasm
diff artifacts/cw_nameservice.wasm download.wasm
```

## 实例化
```bash
# instantiate contract and verify
# 构造函数的参数
INIT='{"purchase_price":{"amount":"100","denom":"upebble"},"transfer_price":{"amount":"999","denom":"upebble"}}'
# 实例化
wasmd tx wasm instantiate $CODE_ID "$INIT" \
    --from wallet --label "awesome name service" $TXFLAG -y --keyring-backend file

# check the contract state (and account balance)
# 再次查询是否有实例化的合约
wasmd query wasm list-contract-by-code $CODE_ID $NODE --output json
CONTRACT=$(wasmd query wasm list-contract-by-code $CODE_ID $NODE --output json | jq -r '.contracts[-1]')
echo $CONTRACT

wasmd query wasm contract $CONTRACT $NODE

# you can dump entire contract state
wasmd query wasm contract-state all $CONTRACT $NODE

# Note that keys are hex encoded, and val is base64 encoded.
# To view the returned data (assuming it is ascii), try something like:
# (Note that in many cases the binary data returned is non in ascii format, thus the encoding)
wasmd query wasm contract-state all $CONTRACT $NODE --output "json" | jq -r '.models[0].key' | xxd -r -ps
wasmd query wasm contract-state all $CONTRACT $NODE --output "json" | jq -r '.models[0].value' | base64 -d

# or try a "smart query", executing against the contract
wasmd query wasm contract-state smart $CONTRACT '{}' $NODE
# (since we didn't implement any valid QueryMsg, we just get a parse error back)
```
## 注册和转移name
```bash
# execute fails if wrong person
REGISTER='{"register":{"name":"fred"}}'
wasmd tx wasm execute $CONTRACT "$REGISTER" \
    --amount 100upebble \
    --from wallet $TXFLAG -y --keyring-backend file

# query name record
NAME_QUERY='{"resolve_record": {"name": "fred"}}'
wasmd query wasm contract-state smart $CONTRACT "$NAME_QUERY" $NODE --output json
# {"data":{"address":"wasm1av9uhya7ltnusvunyqay3xcv9x0nyc872cheu5"}}

# 转移
# buy and transfer name record to wallet2
TRANSFER='{"transfer":{"name":"fred","to":"wasm1um2e88neq8sxzuuem5ztt9d0em033rpr5ps9tv"}}'
wasmd tx wasm execute $CONTRACT "$TRANSFER" \
    --amount 999upebble \
    --from wallet $TXFLAG -y --keyring-backend file
```