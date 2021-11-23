---
title: coswasm 三 入口函数
tags:
  - wasm
  - cosmos
  - 区块链
categories:
  - 区块链
abbrlink: ddae9710
date: 2021-11-23 16:50:41
updated: 2021-11-23 16:50:41
copyright:
---

# cosmwasm 特点
- 合约在`cosmwasm`被称之为`actor`
- 异步栈式调用，将每个调用压栈，触发调用弹栈。避免重入攻击风险，并且有跨链优势

# actor 调用接口
交互`actor`的接口封装在`WasmMsg`结构中，共有5种类型

| 调用接口    | 说明               |
| ----------- | ------------------ |
| Instantiate | 初始化合约入口     |
| Execute     | 执行合约交易       |
| Migrate     | 迁移合约至新的代码 |
| UpdateAdmin | 升级管理员         |
| ClearAdmin  | 清除管理员         |

**说明:**
1. 每个`actor`有个内置的`Admin`充当管理员的角色。
2. 只有`Admin`拥有`UpdateAdmin`、`ClearAdmin`和`Migrate`调用权限。
3. 当`Admin`成功调用`ClearAdmin`后，`actor`将失去管理员角色。
4. 成功调用`Migrate`后，只会改变`actor`的`code`，地址和状态数据不会改变。
5. `Instantiate`、`Execute`和`Migrate`接口，wasm有对应的事件处理函数。

## wasm 调用事件入口
### instantiate
处理初始化事件代码逻辑入口
```rust
#[entry_point]
pub fn instantiate(
  deps: DepsMut,
  env: Env,
  info: MessageInfo,
  msg: InstantiateMsg,
) -> Result<Response, ContractError> {}
```

### execute
处理交互合约事件代码逻辑入口
```rust
#[entry_point]
pub fn execute(
  deps: DepsMut,
  env: Env,
  info: MessageInfo,
  msg: ExecuteMsg,
) -> Result<Response, ContractError> {}
```

### migrate
处理迁移事件代码逻辑入口。  
此部分调用逻辑是在更新的代码中触发。
```rust
#[entry_point]
pub fn migrate(deps: DepsMut, env: Env, msg: MigrateMsg) -> Result<Response, ContractError> {}
```

# actor 查询接口
| 查询接口 | 说明                   |
| -------- | ---------------------- |
| Smart    | 查询公共API的接口      |
| Raw      | 查询kv-store类型的接口 |
## wasm 查询事件入口 
对于`Smart`查询wasm的`query`接口
```rust
#[entry_point]
pub fn query(deps: Deps, env: Env, msg: QueryMsg) -> Result<Binary, ContractError> {}
```
