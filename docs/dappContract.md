---
id: dappContract
title: DAPP合约
sidebar_label: DAPP合约
---

**DAPP合约需要调用的zkRandom接口**

- 创建Item
```solidity
function newItem(
        uint256 projectId,
        address caller,
        address pubkey,
        uint8 model
) returns (uint256 itemId)
```

- 预获取随机数
```
function accept(
        address callback,
        uint256 itemId,
        bytes32 hv
) returns (bytes32 requestKey)
```
- 二次提交获取最终确认的随机数
```
function generateRandom(
        bytes32 requestKey,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
) returns (bytes32 rv)
```

**可选的调用方法**
- 申请解绑
```
function applyUnBond(
        uint256 projectId
)
```

- 提现
```
function withdraw(
        uint256 projectId
)
```

**DAPP合约需要实现的方法**
```
function notify(
        uint item, 
        bytes32 key, 
        bytes32 rv
) returns (bool)
```

# 创建Item

创建Item需要传递传的参数：

- projectId：创建project时生成的ID
- caller：当前合约地址
- pubkey：随机生成的一个以太坊公私钥对的公钥地址，由链下服务器传来
- model：item模式，分为严格模式和宽松模式。严格模式下，同一个item下的随机数需要按顺序消息（即二次提交时需要按照预获取随机数的顺序提交）

严格模式/宽松模式的选择需要DAPP服务商根据自己的使用场景进行选择。

Dapp合约创建Item的伪代码:
```
function createItem(address pubkey, uint8 model) public {
  ...

  uint itemId = ZkRandomContract.newItem(projectId, address(this), pubkey, model)

  ...
}
```
Dapp合约通过对外暴漏createItem方法，供链下服务器调用触发Item的创建。

# 预获取随机数

预获取随机数需要传递的参数：

- callback: 实现了```notify```接口的合约地址在这里为DAPP合约本身
- itemId: 要在哪个Item下获取随机数
- hv: 业务hash，用于异步回调时数据找回

Dapp合约预获取随机数的伪代码:

```
function applyRandom(uint itemId) public {
  ...
  // 组建业务hash
  bytes32 hv = ...;
  bytes32 requestKey = ZkRandomContract.accept(address(this), itemId, hv);
  ...
}
```

Dapp合约对外暴露applyRandom方法，供客户端直接调用。

可以在Dapp合约内部将requestKey和业务数据做关联，以实现通过requestKey找回业务数据。
```
Data[requestKey] = xxData({
  ...
})
```

# 二次提交最终确认

二次提交时需要传递的参数:
- requestKey: 预获取随机数时生成的requestKey
- deadline: 过期事件，单位为秒
- v、r、s: 链下服务器使用Item私钥签名时生成的v、r、s

Dapp合约的伪代码：

```
function generateRandom(bytes32 requestKey, uint deadline, uint8 v, bytes32 r, bytes32 s) public {
  ...
  bytes32 rv = ZkRandomContract.generateRandom(requestKey, deadline, v, r, s);
  ...
}
```

DAPP合约可以直接通过返回值获取到rv，也可以根据notify回调函数获取到rv

```
function notify(uint item, bytes32 requestKey, bytes32 rv) returns (bool) {
  // 找回业务数据
  xxData memory data = Data[requestKey];
  .....
}
```

如果有一次获取多个随机数的需求，可以用下面代码实现:
 ```
 for (uint8 i = 0; i < 10; i++) {
    bytes32 randomVal = keccak256( abi.encode(
            rv,
            i
    ));
    ...
 }

 ```

 # 申请解绑与提现

 在Dapp服务商希望结束使用zkRandom服务时可以申请解绑与提现。申请解绑时，质押的Token将会在锁定1周后释放。提现时，zkRandom会将token转移到合约地址，因此dapp合约还需要写一个提币接口，用于将token从dapp合约中提取到个人账户。