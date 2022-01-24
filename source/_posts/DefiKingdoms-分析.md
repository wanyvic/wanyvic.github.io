---
title: DefiKingdoms 分析
abbrlink: eec96fd2
date: 2022-01-18 10:14:13
updated: 2022-01-21 15:02:32
tags:
categories:
copyright:
---
# 介绍
本次以拆解DefiKingdoms项目的NFT方面为主。

# 关于审计
目前在官方文档中找到关于defi的[审计](https://solidity.finance/audits/DefiKingdoms/)，关于gamefi的审计未找到。

# 关于源码
目前找到关于defi[源码](https://github.com/DefiKingdoms/contracts)，NFT源码未公开、未验证。

# ERC20 

| name         | symbol   | Decimals | address                                                                                                                       |
| ------------ | -------- | -------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Gaia's Tears | DFKTEARS | 0        | [one1yn4q6smd8snq97l7l0n2z6auxpxfv0gyvfd7gr](https://explorer.harmony.one/address/one1yn4q6smd8snq97l7l0n2z6auxpxfv0gyvfd7gr) |
| Jewels       | JEWEL    | 18       | [one1wt93p34l543ym5r77cyqyl3kd0tfqpy0eyd6n0](https://explorer.harmony.one/address/one1wt93p34l543ym5r77cyqyl3kd0tfqpy0eyd6n0) |
| Gold         | DFKGOLD  | 3        | [one18f8deue39azw7qn6elvvyyuz55jejdh8xdfdw2](https://explorer.harmony.one/address/one18f8deue39azw7qn6elvvyyuz55jejdh8xdfdw2) |

# ERC721
NFT：[one1ta6nmn0ekxke427px3npf505w3hadnjuhlh7vv](https://explorer.harmony.one/address/one1ta6nmn0ekxke427px3npf505w3hadnjuhlh7vv)
# 玩法
## Marketplace
交易市场可以交易Token、流动性和买卖部分游戏物品。

### Druid
流动性相关，DEX常规玩法（uniswap）：
1. 创建流动性
2. 添加流动性

合约地址：[one1yjkky5pdr3jje3mggzq3d8gy394vyresl69pgt](https://explorer.harmony.one/address/one1yjkky5pdr3jje3mggzq3d8gy394vyresl69pgt)

### Trader
DEX的swap交易功能相关

### Vendor
使用`DFKGOLD`买卖专业任务获得的部分物品。

合约地址：[one1u5al0rutnxmdx4he8aq6lwv4z95vegkxj3wh82](https://explorer.harmony.one/address/one1u5al0rutnxmdx4he8aq6lwv4z95vegkxj3wh82)
合约交互：
- buyItem 买物品
- 096c5e1a 卖物品

## Gardens
DEX的存入lp挖`JEWEL`

1. 每个Epoch是302400个区块。每个Epoch执行相对应的系数减产。
2. 并且获得的JEWEL按Epoch给的系数部分锁定。
   
[详细参数](https://docs.defikingdoms.com/how-defi-kingdoms-works/the-gardens)

## Bank
存`JEWEL`得到`xJEWEL`类似xsushi。收入来源dex手续费、流动性撤出手续费和游戏手续费。

合约地址：[one1488gx5rasuk9uynnuaz6hn76sjw65e206pmljg](https://explorer.harmony.one/address/one1488gx5rasuk9uynnuaz6hn76sjw65e206pmljg)

合约交互：
- enter 存入
- leave 取出

## Tevern
查看、买卖和出租英雄。本质上是个NFTMarketPlace。购买时抽手续费给dev、bank。

NFTMarketPlace：
- [one1zwn9h8uq883vqv4uqgshrhq9kvxr72yjjjz9la](https://explorer.harmony.one/address/one1zwn9h8uq883vqv4uqgshrhq9kvxr72yjjjz9la)
  
合约交互：
- createAction：设置卖单
- cancelAction：撤销卖单
- bid：购买英雄
  

## Portal
- 召唤需要两个英雄和消耗`DFKTEARS`和`JEWEL`合成召唤水晶，然后调用Open开启水晶得到新的召唤英雄。
- 召唤次数是有固定的限制。并且召唤消耗的`DFKTEARS`随着英雄等级而提高。
- 召唤英雄基因会继承和随机突变上代。

召唤相关合约：
[one17nf6ugpvntj3daltrk66luvm76v6tc648l0sva](https://explorer.harmony.one/address/one17nf6ugpvntj3daltrk66luvm76v6tc648l0sva)

合约交互：
- 租用召唤：c2b40631
- 合成召唤水晶：summonCrystal
- 打开水晶：open

## Professions
任务系统，不同的职业通过任务获得不同的产出：
- 消耗耐力，有几率得到产出。
- 参与职业任务获得`JEWEL`和游戏内资源，以及提升职业技能和经验。
- 当前只有`fisher`和`forager`职业开放，其他职业暂未推出。
- 英雄的基因会影响获得奖励的几率和减少耐力消耗。
- 不同的职业获取的游戏物品不一。
- 游戏物品可以去Vendor交易，也可以用于炼金合成。

Fishing：
- 消耗耐力得到经验和钓鱼的技能提升。有几率得到与钓鱼相关的物品，相关物品可以在Vendor售卖或在Alchemist合成药水。

Foraging：
- 消耗耐力得到经验和觅食的技能提升。有几率得到与植物相关的物品，相关物品可以在Vendor售卖或在Alchemist合成药水。

[one12yqt6vdcygm3zz9q7c7uldjefwv3n6h5trltq4](https://explorer.harmony.one/address/one12yqt6vdcygm3zz9q7c7uldjefwv3n6h5trltq4)

合约交互：
- startQuest 开始任务
- completeQuest 完成任务获得奖励

## Docks
跳转到跨链桥功能。

## Meditation Circle
英雄升级：

当在任务系统中累积足够的经验即可在此升级，需要消耗`JEWEL`和专业符文(专业任务中产出)。也可以消耗水晶提升某属性的成功率。

合约地址：[one1qk2ds6efyvrk5gckat4yu89zsmd2zskpwyytuq](https://explorer.harmony.one/address/one1qk2ds6efyvrk5gckat4yu89zsmd2zskpwyytuq)

合约交互：
- startMeditation 开始升级
- completeMeditation 完成升级

## Alchemist
炼金术师使用多个专业物品合成药水。

# 总结
DefiKingdoms与传统的NFT开盲盒、合成、消耗、抽奖的机制基本无差。相较于同类型游戏玩法稍复杂、可玩性较强。

游戏本质是通过一系列的游戏玩法，设置产出和消耗与门槛使参与游戏的部分玩家获得获利，从而达到财富再分配。