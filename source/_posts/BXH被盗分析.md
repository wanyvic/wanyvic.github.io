---
title: BXH被盗分析
abbrlink: 4b1fd5af
date: 2021-11-03 11:09:50
tags: [区块链, 智能合约, BXH]
categories: [区块链]
copyright:
---
# 观前提示
以下时间均为北京时间
# 参与者
|角色|地址|备注|
|---|---|---| 
|攻击者 Exploiter| [0x48c94305bddfd80c6f4076963866d968cac27d79](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)| BXH攻击者外部账户
|攻击合约 Attacker|[0x887716168FC8c1A2933F97ea3858a94c9E229201](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)| BXH攻击者创建的攻击合约（通过此合约攻击BXH策略）创建于 [2021-10-27 21:47:34](https://bscscan.com/tx/0xf0a833371d843130601dca50553a203e2a07940ec531592cbd3c48003ace34bd) |
|攻击者收款合约|[0x13b81fa9c0873a74c49a85bd8149c1c20bf9d18c](https://bscscan.com/address/0x13b81fa9c0873a74c49a85bd8149c1c20bf9d18c)| 创建于 [2021-10-30 11:15:45](https://bscscan.com/tx/0x6119e86085440083b8e8b6f5435cbc6480df7fde541c8028d013dce7245720a5)
|BXH策略地址|[0x6AcECA12de5a15f11ca51b654433259533B0B802](https://bscscan.com/address/0x6AcECA12de5a15f11ca51b654433259533B0B802)|BXH策略资金地址
|BXH管理员|[0x56146b129017940d06d8e235c02285a3d05d6b7c](https://bscscan.com/address/0x56146b129017940d06d8e235c02285a3d05d6b7c)|总管理员
|BXH访问权限控制合约|[0x83dae57c46985c3a2346e8a3c6991ca908f8b06b](https://bscscan.com/address/0x83dae57c46985c3a2346e8a3c6991ca908f8b06b) | 用于控制各个合约访问权限

# 攻击时序图
{% mermaid sequenceDiagram %}
participant Exploiter as 攻击者 Exploiter
participant Attacker as 攻击合约 Attacker
participant AttackContract as 攻击者收款合约 AttackContract
participant ACL as BXH访问权限控制合约 ACL
participant BXH as BXH策略合约
participant master as BXH管理员
rect rgba(0, 0, 255, .1)
Exploiter->>Attacker: 2021-10-27 21:47:34 创建攻击合约
master->>+ACL: 2021-10-29 16:19:00 给攻击合约提权
ACL-->>-Attacker: 攻击合约获得权限
%% Note over Exploiter,master: 省略:期间尝试七次攻击,均失败
end
Exploiter->>AttackContract: 2021-10-30 11:15:45 创建攻击者收款合约
Exploiter->>+Attacker: 2021-10-30 11:27:47 给攻击者收款合约提权
Attacker-->>+ACL: 访问权限控制合约0xf047601d函数，给攻击者收款合约尝试提权
ACL-->>-AttackContract: ACL内部调用收款合约的0x1f1fcd51。最后给攻击者收款合约添加INTERFACE_ROLE角色
deactivate Attacker
Exploiter->>+Attacker:2021-10-30 11:32:53 第八次发起攻击
Attacker-->>+ACL: 调用ACL的0x8b6b802函数，此函数反编译失败。但猜测内部将资产从策略合约转移给攻击者
ACL-->>+BXH:请求资产转移
BXH-->>-AttackContract:资产转移至攻击者收款合约
deactivate ACL
AttackContract-->>Exploiter:ETH转移至攻击者
deactivate Attacker
{% endmermaid %}
**注:** 攻击者在最后一次成功攻击前，有7次尝试更改逻辑代码的失败攻击。
# 从攻击者角度去分析
1. **2021-10-29 16:19:00** 未知原因拥有BXH最高权限的管理员给Attacker开启`GOVERNANCE_ROLE`权限。
    - 此角色可能是添加删除策略的权限。
    - txid: [0x992c4b9e5cb5c42fbe35403e1e85034fa42beb5bebb3bfa930d1e6dca37742cf](https://bscscan.com/tx/0x992c4b9e5cb5c42fbe35403e1e85034fa42beb5bebb3bfa930d1e6dca37742cf)

2. **2021-10-30 11:27:47** Exploiter给AttackContract添加`INTERFACE_ROLE`权限。
    - 猜测0x13b81fa9c0873a74c49a85bd8149c1c20bf9d18c伪装成策略合约并获得权限。
    - txid: [0x94c3dac0df93d13205f78509dc79363c3c79e4d50f4a5a2a3614ba11e457fa1d](https://bscscan.com/tx/0x94c3dac0df93d13205f78509dc79363c3c79e4d50f4a5a2a3614ba11e457fa1d)

3. **2021-10-30 11:32:53** Exploiter通过`GOVERNANCE_ROLE`权限对ACL合约发起攻击成功获取资产。
    - 此处可能是调用了类似于策略调仓的逻辑，将BXH策略内的资产转移给AttackContract。
    - txid: [0x6466cad9ad300054f0d560db5619a299f67883ad51b6aeafad3061d09674ac84](https://bscscan.com/tx/0x6466cad9ad300054f0d560db5619a299f67883ad51b6aeafad3061d09674ac84)

# ACL内部角色对应的bytes32的值
|ACL内角色|bytes32|
|---|---|
|keccak256('INTERFACE_ROLE')| 0x886f5b5d23a7f8b3645010a8eb98414557d4607145067846682ff187e4950fe3|
|keccak256('WITHDRAW_ROLE')| 0x5d8e12c39142ff96d79d04d15d1ba1269e4fe57bb9d26f43523628b34ba108ec|
|keccak256('GOVERNANCE_ROLE')| 0x71840dc4906352362b0cdaf79870196c8e42acafade72d5d5a6d59291253ceb1|

# 攻击者尝试的8次攻击
## 第一次升级逻辑代码
|调用者|触发者|时间|备注|参数|txid|
|---|---|---|---|---|---|
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-27 21:47:34|攻击者创建攻击合约||[查看](https://bscscan.com/tx/0x6119e86085440083b8e8b6f5435cbc6480df7fde541c8028d013dce7245720a5)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-29 15:58:53|给攻击合约第一次升级逻辑合约[0x2d4d4b79249e6f03cf0e8d672bfc32bdb07981df](https://bscscan.com/address/0x2d4d4b79249e6f03cf0e8d672bfc32bdb07981df)||[查看](https://bscscan.com/tx/0x7e8c28a0af1e70a39b493f32da191d56836f6ce46b7f6fe83af1d066971fa459)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-29 22:26:36|尝试取款失败，参数填的是ETH合约地址|withdraw(`2170ed0880ac9a755fd29b2688956bd959f933f8`)|[查看](https://bscscan.com/tx/0x831f1e2915305abd32eaa7399c17623f3f34fac956d71bedba41cb3469d95241)



## 第二次升级逻辑代码
|调用者|触发者|时间|备注|参数|txid|
|---|---|---|---|---|---|
[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 03:45:16|给攻击合约第二次升级逻辑合约[0x9e47d35f352d322670e5c6def2174d1ae739b3ca](https://bscscan.com/address/0x9e47d35f352d322670e5c6def2174d1ae739b3ca)||[查看](https://bscscan.com/tx/0x7467e536acae21b407bc2e0ca4e4522208dce94830eadd5a1a349517dc3632ae)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 03:51:19|尝试取款失败|withdraw()|[查看](https://bscscan.com/tx/0x4c4c55bca50687487b12c1b2b2e5d9ec8900514d09639cd1a35174f4cb154404)


## 第三次升级逻辑代码
|调用者|触发者|时间|备注|参数|txid|
|---|---|---|---|---|---|
[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 04:01:10|给攻击合约第三次升级逻辑合约[0x977171a577cf954663457af5a2bdc7b1a9951e3b](https://bscscan.com/address/0x977171a577cf954663457af5a2bdc7b1a9951e3b)||[查看](https://bscscan.com/tx/0x332ccad03edf5dd679101595e7b6f1cd3afba6f9f225ed57fda832081448aca7e)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 04:04:35|尝试取款失败|withdraw()|[查看](https://bscscan.com/tx/0x30f837e901e72af2541dd42b029f438ca0845dbc69f81c7e179665b0686be4e0)

## 第四次升级逻辑代码
|调用者|触发者|时间|备注|参数|txid|
|---|---|---|---|---|---|
[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 04:23:39|给攻击合约第四次升级逻辑合约[0x477c53ac688f27b1d2460d85aa681eda9edefde3](https://bscscan.com/address/0x477c53ac688f27b1d2460d85aa681eda9edefde3)||[查看](https://bscscan.com/tx/0x5d8df759d18226781ef1e139e3d1c59862ac9298e4c7a467412325025af6bc25)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 04:24:54|尝试取款失败|withdraw()|[查看](https://bscscan.com/tx/0x4402968870e1e08d83eba0f79e7822720a8aa3e6fd36fbc5fe0120c6f3fa701c)

## 第五次升级逻辑代码
|调用者|触发者|时间|备注|参数|txid|
|---|---|---|---|---|---|
[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 04:34:03|给攻击合约第五次升级逻辑合约[0xe18e97ea16b86f37e335ff1e6d6e96cf74b5f842](https://bscscan.com/address/0xe18e97ea16b86f37e335ff1e6d6e96cf74b5f842)||[查看](https://bscscan.com/tx/0xa268fa121d91eeabc59c4935d0f835f22c7123e6b43c06a0789190658821e5dd)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 04:34:51|尝试取款失败|withdraw()|[查看](https://bscscan.com/tx/0x33ba0953f98cc98dc0ef71113fa8f1512946db7df497877566669d86463b27c5)

## 第六次升级逻辑代码
|调用者|触发者|时间|备注|参数|txid|
|---|---|---|---|---|---|
[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 05:14:09|给攻击合约第六次升级逻辑合约[0x114587850bbdd67f69517143106a9c3af7a9d61c](https://bscscan.com/address/0x114587850bbdd67f69517143106a9c3af7a9d61c)||[查看](https://bscscan.com/tx/0x760ed9d86ec2846430cb2965c17a37c1908caac6848244f6b0f1233bbc585cf0)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 05:16:24|第一次尝试取款至`0x61e365f6031e3cc859911a75c467d3cf2d037e01`失败|withdraw1()|[查看](https://bscscan.com/tx/0xbd0ee9292a47285f723f05d4b40f4f778d5a21e8c5a796719ab90c894173b486)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 09:19:00|第二次尝试取款至`0x61e365f6031e3cc859911a75c467d3cf2d037e01`失败|withdraw1()|[查看](https://bscscan.com/tx/0x57ed3b940d236fd52a1031148fe7ad83c7d676de3466ce8847222678b93b219e)

## 第七次升级逻辑代码
|调用者|触发者|时间|备注|参数|txid|
|---|---|---|---|---|---|
[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 09:36:12|给攻击合约第七次升级逻辑合约[0x22952a0eb4ebd12a1f2d6c318704420f22a2c3db](https://bscscan.com/address/0x22952a0eb4ebd12a1f2d6c318704420f22a2c3db)||[查看](https://bscscan.com/tx/0x065447e1fe4a8d69e660af3570bf59340fdfd51eb1ecbed971472256dcc07cdb)

## 第八次升级逻辑代码
|调用者|触发者|时间|备注|参数|txid|
|---|---|---|---|---|---|
[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 09:39:21|给攻击合约第八次升级逻辑合约[0x114587850bbdd67f69517143106a9c3af7a9d61c](https://bscscan.com/address/0x114587850bbdd67f69517143106a9c3af7a9d61c)||[查看](https://bscscan.com/tx/0x38327b97955cd7f7151b6b4309888a38eb20cd27aada8d23364d6fa3fa350def)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 09:41:09|第一次取款至`0x22952a0eb4ebd12a1f2d6c318704420f22a2c3db`失败|withdraw1()|[查看](https://bscscan.com/tx/0x39559c0e38dd4230e330ed585cb5761bf84bedfc76de575db04f40ff7355d564)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 09:54:53|第二次取款至`0x22952a0eb4ebd12a1f2d6c318704420f22a2c3db`失败|withdraw1()|[查看](https://bscscan.com/tx/0xf0434705664ae4a55b29a3666ae2673bc9a8852f43070c502ebcca7521d3d043)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 10:56:37|第三次取款至`0x91ecadeab1a43ed918309175bcc877ea456378c0`失败|withdraw2()|[查看](https://bscscan.com/tx/0xa4ce16f4ddcbb7eab09d8b9e4d943edcf3f714579174c739aab94a33a54e0fed)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 11:07:00|第四次尝试，此次操作成功给`0x22`地址提权，但未能成功取出存款|withdraw1(`0x22952a0eb4ebd12a1f2d6c318704420f22a2c3db`)|[查看](https://bscscan.com/tx/0xa50946e61a76a91ba0c4a9147c31a63f3285bebe5e935141833c04df27834085)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 11:18:21|第五次取款至`0x13b81fa9c0873a74c49a85bd8149c1c20bf9d18c`未能成功|withdraw1()|[查看](https://bscscan.com/tx/0x3bd6823d3304dd046aadc26c70f5c42f8c0ce84e1991346d1a5a0521848e118a)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 11:19:45|第六次取款至`0x13b81fa9c0873a74c49a85bd8149c1c20bf9d18c`未能成功|withdraw2()|[查看](https://bscscan.com/tx/0x843121bb60581a4f50143a34e752ce8acfbc4304a67f26b3e9bc895ffdea842f)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 11:27:47|第七次尝试，此次操作成功给`0x13b81`地址提权，但未能成功取出存款|withdraw1(`0x13b81fa9c0873a74c49a85bd8149c1c20bf9d18c`)|[查看](https://bscscan.com/tx/0x94c3dac0df93d13205f78509dc79363c3c79e4d50f4a5a2a3614ba11e457fa1d)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 11:29:17|第八次取款至`0x13b81fa9c0873a74c49a85bd8149c1c20bf9d18c`未能成功|withdraw2(`0x13b81fa9c0873a74c49a85bd8149c1c20bf9d18c`)|[查看](https://bscscan.com/tx/0x6696cfbe7bf491f24abeae5a9d5fc2e20d65a1bfb2cd072404c554a3e18879a8)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 11:32:53|第九次取款至`0x13b81fa9c0873a74c49a85bd8149c1c20bf9d18c`成功，大量资产被转移|withdraw2(`0x13b81fa9c0873a74c49a85bd8149c1c20bf9d18c`)|[查看](https://bscscan.com/tx/0x6466cad9ad300054f0d560db5619a299f67883ad51b6aeafad3061d09674ac84)
|[Exploiter](https://bscscan.com/address/0x48c94305bddfd80c6f4076963866d968cac27d79)|[Attacker](https://bscscan.com/address/0x887716168FC8c1A2933F97ea3858a94c9E229201)|2021-10-30 11:36:11|第十次取款至`0x13b81fa9c0873a74c49a85bd8149c1c20bf9d18c`成功，但是已无剩余资产|withdraw2(`0x13b81fa9c0873a74c49a85bd8149c1c20bf9d18c`)|[查看](https://bscscan.com/tx/0xf27301bf6c824c5dc11eb751e0992a7dfeadf01dcc5350ae86a713d4cf9590f3)
