---
title: Cario 笔记 一
abbrlink: fe727175
date: 2022-01-10 11:11:26
updated: 2022-01-17 17:31:55
tags: [cario, 区块链, startnet]
categories: [区块链]
copyright:
---
# 安装
**依赖：ubuntu、python3.7+**
```bash
sudo apt install -y libgmp3-dev python3-dev
python3 -m venv ~/cairo_venv
source ~/cairo_venv/bin/activate
```
确保 venv 已激活 - 您应该(cairo_venv)在命令行提示符中看到。
```bash
pip3 install ecdsa fastecdsa sympy cairo-lang
```

# 编译和运行 Cairo 程序
## 创建一个名为`test.cairo`的文件
其中包含以下几行：
```
func main():
    [ap] = 1000; ap++
    [ap] = 2000; ap++
    [ap] = [ap - 2] + [ap - 1]; ap++
    ret
end
```
## 编译
```bash
cairo-compile test.cairo --output test_compiled.json
```
## 运行
```bash
cairo-run \
  --program=test_compiled.json --print_output \
  --print_info --relocate_prints
```
## 调试
您可以通过提供--tracer标志来打开开罗跟踪器cairo-run。然后在 http://localhost:8100/ 打开它。

# 编写第一份合约
StarkNet 是一个无需许可的去中心化 ZK-Rollup，作为以太坊上的 L2 网络运行，任何 dApp 都可以在其中实现无限规模的计算，而不会影响以太坊的可组合性和安全性。

**主网上的 StarkNet Alpha**  
StarkNet 核心合约：[0xc662c410C0ECf747543f5bA90660f6ABeBD9C8c4](https://etherscan.io/address/0xc662c410C0ECf747543f5bA90660f6ABeBD9C8c4)

定序器网址：https://alpha-mainnet.starknet.io

代码参考：https://github.com/xJonathanLEI/starknet-l1-contracts

**Goerli 上的 StarkNet Alpha 版本 4**  
StarkNet 核心合约：[0xde29d060D45901Fb19ED6C6e959EB22d8626708e](https://etherscan.io/address/0xde29d060D45901Fb19ED6C6e959EB22d8626708e)

音序器网址：https://alpha4.starknet.io

## 让我们从下面的 StarkNet 合约开始：
### 创建一个名为的文件并将`contract.cairo`合约代码复制到其中。
```
# Declare this file as a StarkNet contract and set the required
# builtins.
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin

# Define a storage variable.
@storage_var
func balance() -> (res : felt):
end

# Increases the balance by the given amount.
@external
func increase_balance{
        syscall_ptr : felt*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(amount : felt):
    let (res) = balance.read()
    balance.write(res + amount)
    return ()
end

# Returns the current balance.
@view
func get_balance{
        syscall_ptr : felt*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}() -> (res : felt):
    let (res) = balance.read()
    return (res)
end
```
`%lang starknet`是声明使用的编译器。因此使用`cairo-compile`将编译失败。应使用`starknet-compile`代替。
### 编译
```bash
starknet-compile contract.cairo \
    --output contract_compiled.json \
    --abi contract_abi.json
```
生成了abi和编译后的json。  
abi里显示有`increase_balance`和`get_balance`两个接口。
### 在 StarkNet 测试网上部署合约
您应该`--network=alpha-goerli`在每次使用时传递，或者`STARKNET_NETWORK`按如下方式设置环境变量：(下文均使用了此环境变量)
```bash
export STARKNET_NETWORK=alpha-goerli
```
部署
```bash
starknet deploy --contract contract_compiled.json
```
得到以下结果：
```bash
Deploy transaction was sent.
Contract address: 0x03d2c69f4b89395b062da026c9fabb396b21883f7eb9c2b5cde4fcbcd8120c3f
Transaction hash: 0x7638d01c1736f6329b13cdfcf298e45e0e1cfd460d1e701a040f3cda3233275
```
`Contract address`: 部署的合约地址  
`Transaction hash`: 交易hash

### 与合约交互
运行以下命令来调用该`increase_balance()`函数。`--address`填对应上文部署的合约地址。
```bash
starknet invoke \
    --address 0x03d2c69f4b89395b062da026c9fabb396b21883f7eb9c2b5cde4fcbcd8120c3f \
    --abi contract_abi.json \
    --function increase_balance \
    --inputs 1234
```
结果应如下所示：
```bash
Invoke transaction was sent.
Contract address: 0x03d2c69f4b89395b062da026c9fabb396b21883f7eb9c2b5cde4fcbcd8120c3f
Transaction hash: 0x159b6df5a1a375c2e08244b0276678ceb4abbb9e18546592ae7865362c85441
```
#### 查询状态
```bash
starknet tx_status --hash 0x159b6df5a1a375c2e08244b0276678ceb4abbb9e18546592ae7865362c85441
```
状态可能会经过`PENDING`最终到达`ACCEPTED`:
```bash
{
    "block_hash": "0x34cb1372fa583d5d08eaeba9ba6a9b8fa791df2184b143307b07a08b952db9a",
    "tx_status": "ACCEPTED_ON_L2"
}
```
可能的状态是：
- `NOT_RECEIVED`：尚未收到交易（即未写入存储）。
- `RECEIVED`：事务已被定序器接收。
- `PENDING`：交易通过验证，进入待处理区块。
- `REJECTED`：交易验证失败，因此被跳过。
- `ACCEPTED_ON_L2`：交易通过验证并进入实际创建的区块。
- `ACCEPTED_ON_L1`：交易在链上被接受。
#### 查询余额
使用以下命令查询当前余额：
```bash
starknet call \
    --address 0x03d2c69f4b89395b062da026c9fabb396b21883f7eb9c2b5cde4fcbcd8120c3f \
    --abi contract_abi.json \
    --function get_balance
```
结果应该是：
```bash
1234
```
请注意，要查看最新余额，您应该等到increase_balance 交易状态至少为ACCEPTED_ON_L2（即ACCEPTED_ON_L2或ACCEPTED_ON_L1）。否则，您将看到increase_balance交易执行前的余额（即 0）。
#### 获取交易
```
starknet get_transaction --hash 0x159b6df5a1a375c2e08244b0276678ceb4abbb9e18546592ae7865362c85441
```
```
starknet get_transaction_receipt --hash 0x159b6df5a1a375c2e08244b0276678ceb4abbb9e18546592ae7865362c85441
```
#### 获取代码
```
starknet get_code --contract_address 0x03d2c69f4b89395b062da026c9fabb396b21883f7eb9c2b5cde4fcbcd8120c3f
```
#### 获取块
```
starknet get_block --number 41301
```
#### 通过key来查询state存储的值
编写以下python代码查询key值。
此处对应`@storage_var`的`balance`。
```python
from starkware.starknet.public.abi import get_storage_var_address

balance_key = get_storage_var_address('balance')
print(f'Balance key: {balance_key}')
```

你应该得到：
```bash
Balance key: 916907772491729262376534102982219947830828984996257231353398618781993312401
```
现在，您可以使用以下方法查询余额：
```bash
starknet get_storage_at \
    --contract_address 0x03d2c69f4b89395b062da026c9fabb396b21883f7eb9c2b5cde4fcbcd8120c3f \
    --key 916907772491729262376534102982219947830828984996257231353398618781993312401
```
使用我们目前使用的同一个合约，你应该得到：
```bash
0x4d2
```
请注意，这与调用`get_balance`获得的结果相同。

也可以添加--block_hash指定特定块高查询的结果。
```bash
starknet get_storage_at \
    --contract_address 0x03d2c69f4b89395b062da026c9fabb396b21883f7eb9c2b5cde4fcbcd8120c3f \
    --key 916907772491729262376534102982219947830828984996257231353398618781993312401 \
    --block_hash 0x109decabe47793bead0be4ba2f06835e1b4ede7575cb60b6bd7d29762b3d34e
```
结果：
```bash
0x0
```
### 修改合约使每个人有独立的`balance`
#### 存储映射
使用kvmap存数据
```
# A map from user (public key) to a balance.
@storage_var
func balance(user : felt) -> (res : felt):
end
```
#### 验证签名
我们现在必须修改`increase_balance`以执行以下操作：
- 不同用户写入相应的balance条目。
- 验证用户是否对此次更改操作签名（ACL）。

我们将需要ecdsa内置程序来验证签名，因此我们将%builtins 行更改为：
```
%builtins pedersen range_check ecdsa
```

并添加以下导入语句：
```
from starkware.cairo.common.cairo_builtins import (
    HashBuiltin, SignatureBuiltin)
from starkware.cairo.common.hash import hash2
from starkware.cairo.common.signature import (
    verify_ecdsa_signature)
from starkware.starknet.common.syscalls import get_tx_signature
```
接下来，我们将代码更改`increase_balance()`为：

Increases the balance of the given user by the given amount.
```
@external
func increase_balance{
        syscall_ptr : felt*, pedersen_ptr : HashBuiltin*,
        range_check_ptr, ecdsa_ptr : SignatureBuiltin*}(
        user : felt, amount : felt):
    # Fetch the signature.
    let (sig_len : felt, sig : felt*) = get_tx_signature()

    # Verify the signature length.
    assert sig_len = 2

    # Compute the hash of the message.
    # The hash of (x, 0) is equivalent to the hash of (x).
    let (amount_hash) = hash2{hash_ptr=pedersen_ptr}(amount, 0)

    # Verify the user's signature.
    verify_ecdsa_signature(
        message=amount_hash,
        public_key=user,
        signature_r=sig[0],
        signature_s=sig[1])

    let (res) = balance.read(user=user)
    balance.write(user, res + amount)
    return ()
end
```
`verify_ecdsa_signature`行为类似于断言——如果签名无效，该函数将恢复整个交易。

**请注意，我们在这里没有设置重放保护——一旦用户签署交易，有人可能会多次调用它。防止重放攻击的一种方法是向`increase_balance`中添加一个`nonce`参数，将签名消息更改为随机数和数量的Pedersen哈希，并定义另一个从签名消息到标志（0 或 1）的存储映射，指示是否交易由系统执行。StarkNet的未来版本将处理用户身份验证并防止重放攻击。**

同样，更改`get_balance()`. 这里我们不需要验证签名（因为 StarkNet 的存储无论如何都不是私有的），所以更改更简单：

#### 更改`get_balance()`
```
@view
func get_balance{
        syscall_ptr : felt*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(user : felt) -> (res : felt):
    let (res) = balance.read(user=user)
    return (res)
end
```
#### 编译和部署
将新合同文件另存为`user_auth.cairo`
```
starknet-compile user_auth.cairo \
    --output user_auth_compiled.json \
    --abi user_auth_abi.json

starknet deploy --contract user_auth_compiled.json
```
得到：
```
Deploy transaction was sent.
Contract address: 0x00a22e04b15bac3e899d500ed13ef411ac7b8ceab2711a286abcc1c332f1562a
Transaction hash: 0x6f6d8f253dfaef85398cac92bbd784f43c5f349908cd63cc24f8bcc6056b8
```
#### 与合约交互
需要一对公私钥进行签名，使用python生成：
```python
from starkware.crypto.signature.signature import (
    pedersen_hash, private_to_stark_key, sign)
private_key = 123456 #实际中私钥应该尽量复杂
message_hash = pedersen_hash(4321) # 4321是调用传入的amount
public_key = private_to_stark_key(private_key)
signature = sign(
    msg_hash=message_hash, priv_key=private_key)
print(f'Public key: {public_key}')
print(f'Signature: {signature}')
```
得到：
```
Public key: 3512654880572580671014088124487384125967296770469815068887364768195237224797
Signature: (108302296121512525602232466769839751031019513985679519327892998945991838582, 2367704193695023621669265858663986904236455946141267548614905838978060564822)
```
##### 更新余额
```
starknet invoke \
    --address 0x00a22e04b15bac3e899d500ed13ef411ac7b8ceab2711a286abcc1c332f1562a \
    --abi user_auth_abi.json \
    --function increase_balance \
    --inputs \
        3512654880572580671014088124487384125967296770469815068887364768195237224797 \
        4321 \
    --signature \
        108302296121512525602232466769839751031019513985679519327892998945991838582 \
        2367704193695023621669265858663986904236455946141267548614905838978060564822
```
得到:
```
Invoke transaction was sent.
Contract address: 0x00a22e04b15bac3e899d500ed13ef411ac7b8ceab2711a286abcc1c332f1562a
Transaction hash: 0x2fe59707d58ce8cd3ac10372979a80453a0ab78dbe00646738ffa55137f5ece
```
##### 查询状态和结果
```
starknet tx_status --hash 0x2fe59707d58ce8cd3ac10372979a80453a0ab78dbe00646738ffa55137f5ece
```
```
starknet call \                                                   
    --address 0x00a22e04b15bac3e899d500ed13ef411ac7b8ceab2711a286abcc1c332f1562a \
    --abi user_auth_abi.json \
    --function get_balance \
    --inputs 3512654880572580671014088124487384125967296770469815068887364768195237224797
```
最终`tx_status`改为`ACCEPTED`，`get_balance`改为4321

通过`get_storage_var_address`需要传入用户的pubkey
```
from starkware.starknet.public.abi import get_storage_var_address

user = 3512654880572580671014088124487384125967296770469815068887364768195237224797
user_balance_key = get_storage_var_address('balance', user)
print(f'Storage key for user {user}:\n{user_balance_key}')
```
##### 如果使用无效签名
`tx_status`将会`REJECTED`，
使用`--contract`和`--error_message`回溯错误
```
starknet tx_status \
    --hash TX_HASH \
    --contract user_auth_compiled.json \
    --error_message
```
### 构造函数
使用`@constructor` 声明
```
@constructor
func constructor{
        syscall_ptr : felt*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(owner_address : felt):
    owner.write(value=owner_address)
    return ()
end
```
在deploy时加`--inputs`
```
starknet-compile ownable.cairo \
    --output ownable_compiled.json \
    --abi ownable_abi.json
starknet deploy --contract ownable_compiled.json --inputs 123
```

### 更多功能
1. 多个值的存储变量
```
# A mapping from user to a pair (min, max).
@storage_var
func range(user : felt) -> (res : (felt, felt)):
end

# 读取和写入
@external
func extend_range{
        syscall_ptr : felt*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(user : felt):
    let (min_max) = range.read(user)
    range.write(user, (min_max[0] - 1, min_max[1] + 1))
    return ()
end
```
2. 结构参数的存储变量
```
# 存储变量的参数也可以是结构体或元组，只要它们不包含指针
struct User:
    member first_name : felt
    member last_name : felt
end

# A mapping from a user to 1 if they voted and 0 otherwise.
@storage_var
func user_voted(user : User) -> (res : felt):
end

@external
func vote{
        syscall_ptr : felt*, pedersen_ptr : HashBuiltin*,
        range_check_ptr}(user : User):
    user_voted.write(user, 1)
    return ()
end
```
3. 外部函数使用数组作为参数

在外部函数中使用数组参数的StarkNet合约必须内置`range_check`，用于验证数组的长度是否为非负。
```
@external
func compare_arrays(
        a_len : felt, a : felt*, b_len : felt, b : felt*):
    assert a_len = b_len
    if a_len == 0:
        return ()
    end
    assert a[0] = b[0]
    return compare_arrays(
        a_len=a_len - 1, a=a + 1, b_len=b_len - 1, b=b + 1)
end
```
4. 传入参数包含数组长度和元素
```
starknet invoke \
    --address CONTRACT_ADDRESS \
    --abi contract_abi.json \
    --function compare_arrays \
    --inputs 4 10 20 30 40 2 50 60
```

5. 在 calldata 中传递元组和结构
Calldata 参数和返回值可以是不包含指针的任何类型。例如，带有毛毡成员的结构、毛毡元组和毛毡元组元组。例如：
```
struct Point:
    member x : felt
    member y : felt
end

@view
func sum_points(points : (Point, Point)) -> (res : Point):
    return (
        res=Point(
        x=points[0].x + points[1].x,
        y=points[0].y + points[1].y))
end
```
为了sum_points使用点调用，您应该将以下输入传递给：(1, 2), (10, 20)starknet call
```
starknet call \
    --address CONTRACT_ADDRESS \
    --abi contract_abi.json \
    --function sum_points \
    --inputs 1 2 10 20
```

### 合约相互调用
编写合约`test_proxy.cario`调用之前部署的合约：
```
%lang starknet
%builtins pedersen range_check

from starkware.cairo.common.cairo_builtins import HashBuiltin

@contract_interface
namespace IBalanceContract:
    func increase_balance(amount : felt):
    end

    func get_balance() -> (res : felt):
    end
end

@external
func call_increase_balance{syscall_ptr : felt*, range_check_ptr}(
        contract_address : felt, amount : felt):
    IBalanceContract.increase_balance(
        contract_address=contract_address, amount=amount)
    return ()
end

@view
func call_get_balance{syscall_ptr : felt*, range_check_ptr}(
        contract_address : felt) -> (res : felt):
    let (res) = IBalanceContract.get_balance(
        contract_address=contract_address)
    return (res=res)
end
```
#### 编译部署
```bash
# 编译
starknet-compile test_proxy.cairo \
    --output test_proxy_compiled.json \
    --abi test_proxy_abi.json

# 部署
starknet deploy --contract test_proxy_compiled.json 

# 合约地址及hash
Deploy transaction was sent.
Contract address: 0x02a828cb2ee25629d597625df71ac41779fc9166624654ec52bcc51e77ae75a6
Transaction hash: 0x63e09a708a40d9a6c2e6f33f19d451eb0c774fc2c37db1fa8fe2245c5117c0c
```
#### 调用
```
starknet invoke \
    --address 0x02a828cb2ee25629d597625df71ac41779fc9166624654ec52bcc51e77ae75a6 \
    --abi test_proxy_abi.json \     
    --function call_increase_balance \
    --inputs "0x03d2c69f4b89395b062da026c9fabb396b21883f7eb9c2b5cde4fcbcd8120c3f" 10000

Invoke transaction was sent.
Contract address: 0x02a828cb2ee25629d597625df71ac41779fc9166624654ec52bcc51e77ae75a6
Transaction hash: 0x6fa1501929fdf6971dd35935eb3bdfda2e64668a16d77a665d3555d9992518c
```

#### 查询值变更
```
starknet call \                                                                            
    --address 0x03d2c69f4b89395b062da026c9fabb396b21883f7eb9c2b5cde4fcbcd8120c3f \
    --abi contract_abi.json \
    --function get_balance
11234
```

### 获取调用者地址
get_caller_address()您可以使用库函数检索调用您的函数的合约的地址
```
from starkware.starknet.common.syscalls import get_caller_address

# ...

let (caller_address) = get_caller_address()
```
**注：** 如果调用者是用户则返回0，如果是合约则返回合约地址。

### 获取当前合约的地址
您可以使用get_contract_address()库函数获取当前合约的地址。
```
from starkware.starknet.common.syscalls import (
    get_contract_address)

# ...

let (contract_address) = get_contract_address()
```
以上与`Solidity`中的`address(this)`类似。

# 未完，待续。。。