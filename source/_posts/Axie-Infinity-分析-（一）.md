---
title: Axie Infinity 分析 （一）
abbrlink: 1baceed3
date: 2021-12-17 12:08:26
updated: 2021-12-17 12:08:26
tags: [ eth, 区块链, axie , ronin]
categories: [区块链]
copyright:
---
# 介绍
Axie Infinity是一个卡片对战类型的gamefi游戏玩家可以通过PVE、PVP对战、获取SLP、购买NFT精灵、繁殖、升级等养成类游戏。
部分代码开源：
[public-smart-contracts](https://github.com/axieinfinity/public-smart-contracts)
[ronin-smart-contracts](https://github.com/axieinfinity/ronin-smart-contracts)


# 玩法分析
1. 用户通过跨链将资产转移到ronin chain。需购买或其他方式得到3个NFT小精灵，进行游戏对战。
2. 用私钥对邮箱地址签名进行账号绑定。
3. 玩家进入游戏可以进行PVE、PVP获得SLP。用户等待锁定期后通过链上claim获取SLP。SLP的释放速率受控于项目方。
4. NFT可以有性繁殖，中间消耗SLP和AXS。
5. NFT交易市场可以自由买卖、拍卖NFT。
6. 玩家获取的SLP和AXS可以进行Defi挖矿（Lp挖矿和单币挖矿）。
7. 对于PVP Rank topN，赛季后会释放奖金激励。

# 涉及的合约
## ronin侧
###  Ronin Gateway Contract
Ronin和Ethereum的跨链桥
#### 合约地址
[ronin:e35d62ebe18413d96ca2a2f7cf215bb21a406b4b](https://explorer.roninchain.com/address/ronin:e35d62ebe18413d96ca2a2f7cf215bb21a406b4b)
#### 作用
Ronin和Ethereum的跨链桥
#### 核心方法
| functionId                                                                                                     | 调用权限      | 备注                                                                               |
| -------------------------------------------------------------------------------------------------------------- | ------------- | ---------------------------------------------------------------------------------- |
| acknowledWithdrawalOnMainchain(uint256 _withdrawalId)                                                          | onlyValidator | 需要多个validator对取款到ethereum的确认,如果阈值达到设置，则从pending中移除。      |
| submitWithdrawalSignatures(uint256 _withdrawalId,bool _shouldReplace,bytes memory _sig)                        | onlyValidator | validator提供取款操作签名给ethereum，当ethereum收集足够的签名时可以被批准取出token |
| withdrawERC20(address _token, uint256 _amount)                                                                 | user          | 用户调用取token到ethereum，创建withdrawEntry到pending                              |
| depositERCTokenFor(uint256 _depositId, address _owner, address _token, uint32 _standard, uint256 _tokenNumber) | onlyValidator | validator调用，当满足阈值时可以在ronin取出token                                    |

#### 提款到MainChain
{% mermaid sequenceDiagram %}
participant Validitor
participant User
participant SideChain as SideChain Gateway
participant MainChain as MainChain Gateway

User->>+SideChain:withdrawERC20 (transferFrom token)
Validitor->>SideChain:N x acknowledWithdrawalOnMainchain
Validitor->>SideChain:N x submitWithdrawalSignatures
SideChain-->>-User: N x Signatures
User->>+MainChain:withdrawERC20 (with Signatures)
MainChain-->>-User:token transfer

{% endmermaid %}

#### 存款到SideChain
{% mermaid sequenceDiagram %}
participant Validitor
participant User
participant SideChain as SideChain Gateway
participant MainChain as MainChain Gateway

User->>+MainChain:depositERC20
MainChain-->>-Validitor:emit TokenDeposited
Validitor->>+SideChain:N x depositERCTokenFor
SideChain-->>-User:token transfer

{% endmermaid %}


**注：涉及weth取出存入与token操作相同此处不再赘述**  
**注：涉及batch操作与原子操作类似此处不再赘述**

###  Ronin Registry Contract
#### 合约地址
[ronin:3a860626b0467809d50c58bef89b8ac6247fd62a](https://explorer.roninchain.com/address/ronin:3a860626b0467809d50c58bef89b8ac6247fd62a)
#### 作用
维护例如GATEWAY、WETH_TOKEN、VALIDATOR、ACKNOWLEDGEMENT合约名称到地址的注册合约。

###  Ronin Validator Contract
#### 合约地址
[ronin:0000000000000000000000000000000000000011](https://explorer.roninchain.com/address/ronin:0000000000000000000000000000000000000011)
#### 作用
validator白名单合约，用于添加、删除validator和更改阈值投票。维护Ronin的POA共识。

###  Smooth Love Potion Contract
#### 合约地址
[ronin:a8754b9fa15fc18bb59458815510e40a12cd2014](https://explorer.roninchain.com/address/ronin:a8754b9fa15fc18bb59458815510e40a12cd2014)
#### 作用
游戏内的ERC20：SLP，用户进行游戏后服务器发放给玩家的“经验”，用于其他玩法消耗。
#### 核心方法
除常规IERC20外，多了一个checkpoint方法用于“claim”。

| function                                                                       | 调用权限 | 备注                                    |
| ------------------------------------------------------------------------------ | -------- | --------------------------------------- |
| checkpoint(address _owner,uint256 _amount,uint256 _createdAt,bytes _signature) | user     | 用户取出slp时需要服务器给出相对应的签名 |

###  Axie Infinity Shard Contract
#### 合约地址
[ronin:97a9107c1793bc407d6f527b77e7fff4d812bece](https://explorer.roninchain.com/address/ronin:97a9107c1793bc407d6f527b77e7fff4d812bece)
#### 作用
ERC20合约：axs

###  Axie Contract
#### 合约地址
[ronin:32950db2a7164ae833121501c797d79e7b79d74c](https://explorer.roninchain.com/address/ronin:32950db2a7164ae833121501c797d79e7b79d74c)
#### 作用
ERC721合约：axie。除了正常的IERC721以外，里面包含了一些管理员操作例如：growAxieggToAdult、batchMintAxieggs、mintAxie、和一些改变genes的设置。此部分代码未开源。


### Katana Router Contract
#### 合约地址
[ronin:7d0556d55ca1a92708681e2e231733ebd922597d](https://explorer.roninchain.com/address/ronin:7d0556d55ca1a92708681e2e231733ebd922597d)
#### Lp对
slp-weth lp [ronin:306a28279d04a47468ed83d55088d0dcd1369294](https://explorer.roninchain.com/address/ronin:306a28279d04a47468ed83d55088d0dcd1369294)  
axs-weth lp [ronin:c6344bc1604fcab1a5aad712d766796e2b7a70b9](https://explorer.roninchain.com/address/ronin:c6344bc1604fcab1a5aad712d766796e2b7a70b9)  
usdc-weth lp [ronin:a7964991f339668107e2b6a6f6b8e8b74aa9d017](https://explorer.roninchain.com/address/ronin:a7964991f339668107e2b6a6f6b8e8b74aa9d017)



### AXS Staking Pool Contract

#### 合约地址
[ronin:05b0bb3c1c320b280501b86706c3551995bc8571](https://explorer.roninchain.com/address/ronin:05b0bb3c1c320b280501b86706c3551995bc8571)
#### 作用
质押axs挖矿（单币挖矿）
#### 核心方法
| function              | 备注     |
| --------------------- | -------- |
| stake(uint256)        | 质押axs  |
| restakeRewards()      | 复投     |
| claimPendingRewards() | 获取收益 |
| unstake(uint256)      | 取出质押 |

### Staking Pool Contracts

#### 合约地址
SLP-WETH LP Staking Pool [ronin:d4640c26c1a31cd632d8ae1a96fe5ac135d1eb52](https://explorer.roninchain.com/address/ronin:d4640c26c1a31cd632d8ae1a96fe5ac135d1eb52)
AXS-WETH LP Staking Pool [ronin:487671acdea3745b6dac3ae8d1757b44a04bfe8a](https://explorer.roninchain.com/address/ronin:487671acdea3745b6dac3ae8d1757b44a04bfe8a)

#### 作用
质押LP挖矿。

### Marketplace Contract
NFT交易二级市场

#### 合约地址
[ronin:213073989821f738a7ba3520c3d31a1f9ad31bbd](https://explorer.roninchain.com/address/ronin:213073989821f738a7ba3520c3d31a1f9ad31bbd)

#### 核心方法
| function                                                                           | 备注                     |
| ---------------------------------------------------------------------------------- | ------------------------ |
| settleAuction(address,address,uint256,uint256,uint256)                             | 买NFT                    |
| createAuction(uint8[],address[],uint256[],uint256[],uint256[],address[],uint256[]) | 创建卖单                 |
| cancelAuction(uint256)                                                             | 撤销卖单                 |
| revalidateRelatedAuctions(uint256)                                                 | 未知，似乎是管理员的操作 |

{% mermaid sequenceDiagram %}
participant Consumer
participant Seller
participant Marketplace

Seller->>+Marketplace:createAuction 创建卖单
alt
    Seller->>Marketplace:cancelAuction 撤单
else
    Consumer->>Marketplace:settleAuction 购买
    Marketplace-->>-Consumer:transfer NFT
end
{% endmermaid %}

## Ethereum侧
Ethereum侧包含gateway、ERC20、ERC721合约与ronin侧大致相同。不同点在于Ethereum上的gateway使用了EIP712方式取款减少交互节约gas。