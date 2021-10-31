---
title: EarnDefi被盗分析
abbrlink: 28567f46
date: 2021-10-31 13:45:08
tags:
---

# Earn Defi 单币挖矿合约架构

## 入口合约
负责单笔挖矿用户接入的账户系统  

地址：[0x80b0eafa5aaec24c3971d55a4919e8d6a6b71c78](https://hecoinfo.com/address/0x80b0eafa5aaec24c3971d55a4919e8d6a6b71c78)  
管理员：[0x61f942b6eedd9b400aa9780637c84d2770af85b6](https://hecoinfo.com/address/0x61f942b6eedd9b400aa9780637c84d2770af85b6)  

当前账户内剩余USDT数量：
```
282,095.648425891624129973 USDTHECO  
248,056,556.021327376723397534 HPT  
3,001.664842435153282252 HLTC  
3,027.460637161641773855 HFIL  
2.528440468150094002 HBTC  
22.021496524943978192 ETH  
3,030.812388128947611371 WHT  
3,041.24777095 HUSD
```

合约内部用map维护了pool列表，其中USDT的pid为2

**交互方法**

|  方法   | MethodID | 备注 | 示例txId |
|  ----  | ----  | ---- | ---- |
|deposit(uint256 _pid, uint256 _amount)  |  0xe2bbb158 | 用户存入指定数量的某种token |  [0xe314a2bc03a274f06bfe25f76c0f3ad1c8beadda5a899846c8a4ef905c64b709](https://hecoinfo.com/tx/0xe314a2bc03a274f06bfe25f76c0f3ad1c8beadda5a899846c8a4ef905c64b709)
|depositAll(uint256 _pid)  |  0xc6f678bd | 用户存入所有的某种token | [0xbd9973852e45cda3839080e8d3c6daff2e9c9db4d5f67eca2b2056719388f7fc](https://hecoinfo.com/tx/0xbd9973852e45cda3839080e8d3c6daff2e9c9db4d5f67eca2b2056719388f7fc)
|withdraw(uint256 _bid, uint256 _refId)  | 0x441a3e70 | 用户提出指定数量的某种token | [0xb4aa7ee92d21a3b40d5ef3f96ec8646f45c1d47a15a1d183cacf2cfc8e43cddb](https://hecoinfo.com/tx/0xb4aa7ee92d21a3b40d5ef3f96ec8646f45c1d47a15a1d183cacf2cfc8e43cddb)
|withdrawAll(uint256 _pid)  | 0x958e2d31 | 用户提出所有的某种token | [0x94e889b92424a9b879bd01ff3d762ba3580b0993e8bf1089566b79639f8aadcf](https://hecoinfo.com/tx/0x94e889b92424a9b879bd01ff3d762ba3580b0993e8bf1089566b79639f8aadcf)
|pause?(bool) |0xe83ddd87| 管理员可以设置参数暂停存提,true为暂停，false为开放。函数名未确定 |[0x2a9c37e9df8998e4c51f1b7446dee1baeddbb5c087c4687d4db4bc992f4470d2](https://hecoinfo.com/tx/0x2a9c37e9df8998e4c51f1b7446dee1baeddbb5c087c4687d4db4bc992f4470d2)
|addStrategy(address addr)| 0x223e5479 | 添加币种| [0x7e957b90142617ead7d349456590c1a262ad2eba2727aa4c503ffd2fe3e1ff83](https://hecoinfo.com/tx/0x7e957b90142617ead7d349456590c1a262ad2eba2727aa4c503ffd2fe3e1ff83)
|harvestAll() |0x8ed955b9 | | [0x7a5dc22804e51ff0a692b6f3ee0e52069d5fc07c0b1ed4b8f138383ac68065ac](https://hecoinfo.com/tx/0x7a5dc22804e51ff0a692b6f3ee0e52069d5fc07c0b1ed4b8f138383ac68065ac)
|unknowed(address)|0x776d59e4| 貌似是设置对下层合约进行重新投资的函数  |[0x2efeea460127f3f598f3836a08dacafe55c21d8ba1e99bacff5425f38710a54b](https://hecoinfo.com/tx/0x2efeea460127f3f598f3836a08dacafe55c21d8ba1e99bacff5425f38710a54b)

### storage 状态变量

**unknown1526fe27 is array of struct at storage 4**

struct poolInfo[]:
- erc20Address address
- pid?
- blockNumber
- ?
- 已投额度
- 总计额度
```
例如：pid为2的USDT
[
  '0xa71edc38d189767582c38a3145b5873052c3e47a',
  '125',
  '9567307',
  '10918534050',
  '8999854826604473921412569',
  '30000000000000000000000000'
]
```

**unknowna267526b is array of addr at storage 5**
策略列表
```
address[] strategys;
sid: 0, addr: 0x447e8a7f84368a762294be9a8a34f606206bde0e    //boo 涉及wht的投资策略
sid: 1, addr: 0x40b910d008cebfd433154e52ce343ded3d248378    //Depth 涉及 USDT 11 0
sid: 2, addr: 0xbb8f5d4e826c32085baabb3623cffb1887dd5917    //Depth 涉及 HUSD   pid 9 510853.05530818
sid: 3, addr: 0x1843f672f6d60db32b374d0eb3acd0bdd12cd3c1    //boo 涉及HFIL  // 空
sid: 4, addr: 0xc19c9adc5fe02a1d91b2721c2e232e1c37527392    //coinwind 涉及HFIL // 空
sid: 5, addr: 0x996b6e9635f862c834cf50403cd33aacbfa4b207    //coinwind 涉及USDT // 空
sid: 6, addr: 0xadc0a8b9e7149d7c6a5bbecf6a4016576e942247    //coinwind 涉及HUSD //空
sid: 7, addr: 0x4f676ec7a3d75f7be2fcbda74df5a31708822aa5    //bxh 的HDOT    //空
sid: 8, addr: 0x9aaa9ed5eed70494df66e0540d0bc5512c3938c6    //gHLTC         //空
sid: 9, addr: 0x36768a55a260e561291b2fc14f67ddee86fc6fa1    //mdx HT-HUSD   //空
sid: 10, addr: 0xf207e6d9f1e59cfaa8358e849cd6b7c8d323e025   //mdx HT-BTC    //空
sid: 11, addr: 0xa8a8881edc4660b834c2b9e5add3cb46db1d65fb   //mdx ETH-HUSD  //空
sid: 12, addr: 0xb3cd38e467139721b319401e5bc06e3930b2216c   //分配dev收益mdx
sid: 13, addr: 0xf1c11b4ac327d3e2da24114c183ec8a977ebcfe9   //coinwind HBTC //空
sid: 14, addr: 0xe0d179935161af517fbd5c57e993cce9e65c6137   //分配dev收益mdx //空
sid: 15, addr: 0x55297d6f8fb33beafe5880c3b59ba6cce7c8b13c   //coinwind WHT  
sid: 16, addr: 0x954e4a0eb2aabdc5a8140a162f6cab649f9fc0c6   //BOO HUSD
sid: 17, addr: 0xe4fdb79bae96a082a9d716b09f0fa8eedfa5ee32   // MDX HBTC-USDT 8 1.827616583664665540个LP/233065.623498707777205998池内总计
sid: 18, addr: 0x96164d9398408b5dc2cd6705ac7b98b231c26c22   //BOO HBTC  //空
sid: 19, addr: 0x8a2fc39d821916149f4d7b722fec2be5d2abe825   //BOO ETH  //空
sid: 20, addr: 0xc461dd52d605ab22d762d1a019da4342c9d8ad12   //back平台 ETH   //空
sid: 21, addr: 0x54556e1aa1092bc614ad39f519fabce66e74b588   //back平台 HBTC  //空
sid: 22, addr: 0x5948ad44812e02868aa5c9f018943d35670eaf8f   //BOO USDT  //空 
sid: 23, addr: 0xdccf2926ee39de136ee31b805a5bd7f3d8c58119   //BACK WHT //空 
sid: 24, addr: 0x22a411ff3ad2d25fe53ee0143cc36b13f8a9a6cc   //分配dev收益mdx     //空 
sid: 25, addr: 0x8e5dc937013f67e4e506fe2e3d0774b66147404a   //Belt.fi ETH
sid: 26, addr: 0x1fb01f73e3bdfd805bf8c8f39e4885d08369906d   //Belt.fi HBTC
sid: 27, addr: 0xd16a55ba2b84419c83a671d8992db92d2910acd6   //gHLTC
sid: 28, addr: 0xf49a3ea13728e46204900c87390a45365c465ab6   //mdx HBTC-ETH
sid: 29, addr: 0xd3fcca8a912cd975ae85c57c903f1856a147b2b3   //mdx HETH-USDT 9 0
sid: 30, addr: 0xc1bd204bc1d0c1bf239aaf103b9a407f60056fbf   //Belt.fi WHT 有
sid: 31, addr: 0xe2992d7d4f905294639ebe3fd916a2ed4de28a24   //MDX 过度
sid: 32, addr: 0x0ebfe4706f08aea1130e7e7239e06edd45b491d3   //MDX 过度
sid: 33, addr: 0xf02bd1bb3958de33818490f81f3f95a0157dc2cb
sid: 34, addr: 0xecf8b5fd8c1b82f8de88fb061641ad9c291fb885   //FILDA MDX
sid: 35, addr: 0xb4680dbb6a283065d4f45f4c5ab54138182da0ca   //DEMETER HDOT (8185 dmHDOT 304dmt)
sid: 36, addr: 0x4d1fc3664298dc1f0ceae114f16a92ddcb659181   // 管理员调用的dmt
sid: 37, addr: 0x8f89467cdfdd96ce9e35aac6825c1ffadd4a684a   //借贷平台 20,774.53639379 dmUSDT
sid: 38, addr: 0xd77adc07294df8b0d1a95571889a649c12c744b0   //dmMDX 1,051,236.85964317 dmMDX 11,707.917522113808967497 DMT
sid: 39, addr: 0x0298c2b32eae4da002a15f36fdf7615bea3da047   //HUSD                       
```

# 关于审计

|  审计方  | 结果 |
|  ----  | ----  |
|成都链安|只审计了EDC代币的合约|
|CERTIK|只了LP池内相关的2个合约|
|慢雾科技| 审计打不开|

另外：审计报告内的github链接打不开  
[https://github.com/Wendy-Earndefi/vault](https://github.com/Wendy-Earndefi/vault)