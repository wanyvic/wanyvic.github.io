---
title: Star-Atlas、Fancy-Birds和DefiKingdoms 分析
abbrlink: 1b44cf2e
date: 2022-01-12 13:57:00
updated: 2022-01-12 13:57:00
tags:
categories:
copyright:
---

# Star Atlas

官网：https://staratlas.com/

Star Atlas是一个基于2620年的虚拟元宇宙游戏。基于虚幻5开发的3A MMORPG游戏（链游版的EVE）。故事背景设定在 2620 年，在官方设定中，宇宙分为了三大派系：由人类统治的 MUD 领地；外星种族联盟的 ONI 区域；知觉机器人控制的 Ustur 星区。三大派系为资源、领土和统治权进行着持续的战争。玩家可自由加入其中一方，为自己的派系贡献力量。而玩家的行为则将直接影响这场星际冲突的结果，通过在游戏中做出贡献，玩家可获得代币奖励。

特点：
- 资产链上化（NFT）：舰船、船员、宠物、组件、土地、建筑物、服装等等
- 双代币经济体系

## 通证
游戏中也将实行双通证，[ATLAS](https://explorer.solana.com/address/ATLASXmbPQxBUYbxPsV97usA3fPQYEqzQBUHgiFCUsXx)和[POLIS](https://explorer.solana.com/address/poLisWXnNRwC6oBu1vHiuKQzFjGL4XDSu4g9qjz9qVk)。

**ATLAS：** ATLAS将作为游戏内的主要交易媒介，用于购买游戏内的土地、装备、组件等等，总量360亿。

**POLIS：** 项目的治理通证，将用于参与决策游戏内的一些玩法和机制，总量3.6亿。

## 进展
- 第一阶段：OpenSea资产售卖
- 第二阶段：网页迷你小游戏
- 第三阶段：船坞组件售卖
- 第四阶段：最终预售公测
- 第五阶段：持续售卖

当前进展：阶段二，玩家可以在marketplace上自由买卖NFT。关于3A游戏本身暂未看到任何进度。按3A大作开发惯例，一般需要5-10年不等。

## github
当前github主要开源一个与token解锁相关智能合约的仓库 [token-vesting](https://github.com/staratlasmeta/token-vesting)

## 关于dex-Serum
solana上的所有DEX都基于Serum，也就是说只要你知道了某一交易对的Market ID，你可以在任何前端进入交易。用SA举例就是你要买飞船，不用在官网买，直接在支持Serum的任意前端导入market ID，就可以进行交易，查看订单。


# Fancy Birds
官网：https://www.fancybirds.io/

merit-circle投资。玩法类似于2013年大火的`flappy bird`。一共发行8888只创世鸟。拥有繁殖、遗传基因玩法，罕见的基因可以提升打金收入。玩法本质上类似于Axie Infinity。

## 当前进度
- token已发行：FNC
- 2021年12月15日上线质押挖矿(Defi 挖矿玩法)
- 预计2022 Q1上线创世鸟的铸造和marketplace
- 预计2022 Q2上线游戏主体

## github
审计报告中找到的地址，打不开。
似乎未开源：https://github.com/Fancy-Birds/  
审计报告包含Token和staking相关的合约代码审计

## defi 玩法
常规defi玩法，质押FNC或LP挖矿。  
https://staking.fancybirds.io/

## 通证
[Fancy Token($FNC)](https://etherscan.io/token/0x7f280daC515121DcdA3EaC69eB4C13a52392CACE)

[Staked Fancy Games LP](https://etherscan.io/token/0x162ce5530Aba30583cCaF79dD72650CfEB050a23)(锁仓SLP挖矿合约)

[sushi FNC-WETH lp (SLP)](https://etherscan.io/address/0x82ecdd4635766560c4e3a8efa8a7c1fff566111e)

## 游戏模式
- PVP、PVE打金。（flappy bird有类似的辅助软件）
- 交易、繁殖、锦标赛（玩法类似于Axie Infinity）。
- 后续sFNC用于游戏经济中的繁殖、赌注和其他游戏内活动。

## 技术手段猜测
由于代码未开源，现根据游戏玩法对技术的猜测
1. 使用`Immutable X`发行、交易、跨链NFT，和FNC（DAO）。
2. 基于starknet Cairo开发繁殖、PVP、PVE奖金系统。


# DefiKingdoms
https://defikingdoms.com/
DeFi Kingdoms是基于Harmony Protocol区块链开发的一款play2earn游戏。

DeFi Kingdoms旨在将DeFi元素包装成一个成为一个完整的功能生态系统。
本质上还是个DEX+NFT。

## 通证
JEWEL：[0x72cb10c6bfa5624dd07ef608027e366bd690048f](https://explorer.harmony.one/address/0x72cb10c6bfa5624dd07ef608027e366bd690048f)

xJEWEL：[0xa9ce83507d872c5e1273e745abcfda849daa654f](https://explorer.harmony.one/address/0xa9ce83507d872c5e1273e745abcfda849daa654f)

## github
部分合约：https://github.com/DefiKingdoms/contracts/

- [AidropClaim.sol](https://explorer.harmony.one/address/0xa678d193fEcC677e137a00FEFb43a9ccffA53210)： 空投相关的合约很简单，merkeproof都没有用到
- [Bank.sol](https://explorer.harmony.one/address/0xA9cE83507D872C5e1273E745aBcfDa849DAA654F)：质押JEWEL挖矿，使用的xsushi代码base
- [Banker.sol](https://explorer.harmony.one/address/0x3685ec75ea531424bbe67db11e07013abeb95f1e)：一个将合约内部的lp解除流动性并将token转换成`JEWEL`存入Bank合约中。lp来源swap的feeTo。
- [MasterGardener.sol](https://explorer.harmony.one/address/0xDB30643c71aC9e2122cA0341ED77d09D5f99F924)：流动性矿池类似于masterchef
- [UniswapV2Factory.sol](https://explorer.harmony.one/address/0x9014B937069918bd319f80e8B3BB4A2cf6FAA5F7): uniswap的代码base，feeTo设置为Banker
- [UniswapV2Router02.sol](https://explorer.harmony.one/address/0x24ad62502d1C652Cc7684081169D04896aC20f30) 

## NFT
玩家可以通过购买、合成、售卖等方式玩NFT获利。NFT可以影响defi的收益。
- Gen0：2090只创世英雄
- 稀有度：紫、橙、蓝、绿、白
- 英雄职业和专业影响defi的收益
- 特定的职业+特定的专业以及稀有度可以最大化收益产出，玩家主要就是合成此类卡片售卖或挖矿。

### 专业功能未开放
- mining：可以提前解锁JEWEL和黄金（黄金Token未上线）
- Gardening：增加流动性挖矿利率
- Fishing：可以获得宝物
- Foraging：可以炼制素材

### 职业
职业分基础职业、进阶职业、精英职业。最上级职业。

两个职业可以合成上阶职业（有合成率）。

合成次数从职业低到高，有不同的限制，越往上合成次数越少。
