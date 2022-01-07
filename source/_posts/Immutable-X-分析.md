---
title: Immutable X 分析
abbrlink: bed1adf7
date: 2021-12-26 14:40:34
updated: 2021-12-26 14:40:34
tags:
categories:
copyright:
---

# 介绍
Immutable X是一个基于StarkWare的layer2网络使用Cairo编写的NFT跨链和交易应用。用户可以在此网络内进行快速低成本的交易。并且由于StarkNet底层使用了ZK-Rollup的方案，相比传统sidechain拥有更高的去中心化安全性、即时交易确认、高tps、抗DDos等优点。

# 原理
Immutable X使用ZK-Rollup，将layer2上的一定量的交易打包，批量生成有效的证明，提交到layer1上的智能合约进行验证。通过验证后即可完成“跨链”的存取操作。

# Immutable X的使用
## 在Immutable X集成ERC721
**先决条件：**
1. ERC721的部署：在layer1上部署继承了Immutable X的Mintable合约的ERC721。
2. 合约注册：上文所述的合约要在Immutable X登记。
3. 用户注册：用户使用前也需要先登记。目的是为了将layer1私钥与layer2私钥进行绑定。

### ERC721的部署
在layer1上部署继承了Immutable X的Mintable合约的ERC721。确保IMX拥有铸币权。

**Mintable.sol**  
此合约内部提供了一个mintFor方法确保IMX拥有铸币权。token特性相关的代码需要实现_mintFor方法。
```
modifier onlyIMX() {
        require(msg.sender == imx, "Function can only be called by IMX");
        _;
    }
​
    function mintFor(
        address user,
        uint256 quantity,
        bytes calldata mintingBlob
    ) external override onlyIMX {
        require(quantity == 1, "Mintable: invalid quantity");
        (uint256 id, bytes memory blueprint) = Minting.split(mintingBlob);
        _mintFor(user, id, blueprint);
        blueprints[id] = blueprint;
        emit AssetMinted(user, id, blueprint);
    }
​
    function _mintFor(
        address to,
        uint256 id,
        bytes memory blueprint
    ) internal virtual;
```

### 合约注册
部署合约后，需要向 Immutable X 注册。
目前只支持手动验证注册，后续项目方支持自动验证注册。  
将以下详细信息，[在此处](https://support.immutable.com/hc/en-us/requests/new)提交给项目方后收到回执邮件。  
然后在[https://api.x.immutable.com/v1/collections](https://api.x.immutable.com/v1/collections)可以查询到我们的合约。
```
{
  "name": "Collection Name (usually Partner Name)",
  "description": "Some Description of this collection",
  "owner_public_key": "<CHANGE-ME>",
  "contract_address": "<CHANGE-ME>",
  "metadata_api_url": "https://<CHANGE-ME>",
  "icon_url": "https://<CHANGE-ME>",
  "collection_image_url": "https://<CHANGE-ME>"
}
```
### 用户注册
将按照EIP-2645对用户的layer1和layer2的私钥进行绑定。
![](https://files.readme.io/3bc4579-THLcqN4gMFQp.png)

## 在Immutable X使用
### 铸币
![](https://files.readme.io/9b5debc-THLcqN4gMFQp_1.png)

1. 用户在layer2用对应的私钥发起铸币请求。
2. Immutable X Engine对签名、请求、状态等验证同步提交到layer2。
3. layer2最后根据状态转移进行零知识证明生成proof。
4. 将证明结果提交给layer1。

### 资产存款
用户在layer1上发起存款请求，layer2接收到事件后，在layer2上进行铸造。
![](https://files.readme.io/0048928-THLcqN4gMFQp_2.png)
1. 用户首先向API请求存款数据。
2. 用户拿到存款数据后，将token存入layer1的存款合约。
3. 存款合约发出事件后，Immutable X engine收到事件后在layer2上处理存款请求。
4. layer2生成proof提交到layer1。

### 资产提取
用户在layer2上发起提取请求。当确认后，用户可以在layer1上发起请求进行提款。
![](https://files.readme.io/f18d111-THLcqN4gMFQp_3.png)
1. 用户首先向API请求提款数据。
2. 用户拿到提款数据后发起提款。
3. 当layer2协商一致后并且proof成功上链后，layer1用户会收到提款事件。
4. 用户layer1发起提款，并收到资产。

### 资产交易
用户交易本身发生在layer2，无须与layer1交互。
![](https://files.readme.io/6e913c7-THLcqN4gMFQp_4.png)
1. 用户首先向API请求卖单数据。
2. 用户根据数据提交卖单。（此部分没有进行实质上的状态数据变化，所有无须与其他节点同步）
3. 某用户确认认购后，Immutable Engine将卖单买单数据同步给其他节点。
4. 生成proof提交到layer1。

# 参考资料
>Cairo合约开发文档 https://www.cairo-lang.org/  
>immutable x 文档 https://docs.x.immutable.com/docs/  
>starknet https://starkware.co/starknet/