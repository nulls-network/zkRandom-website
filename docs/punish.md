---
id: createItem
title: 创建Item
sidebar_label: 创建Item
---

**创建Item需要调用的接口**

```solidity
function newItem(
        uint256 projectId,
        address caller,
        address pubkey,
        uint8 model
) returns (uint256 itemId)
```

**相关事件**
```
NewItem(uint256 projectId, uint256 itemId, address pubkey, uint8 model);
```

**DAPP合约需要实现的方法**
```
function notify(uint item, bytes32 key, bytes32 rv) returns (bool);
```

# 创建Item流程

1. DAPP开发商随机生成一个以太坊账户公私钥对，并存储起来
2. DAPP
3. 调用zkRandom合约的regist方法，第一个参数填项目名称、第二个参数填项目管理员账户地址（该地址可以是普通账户地址也可以是合约地址）、第三个参数填质押金额（该金额有一个最小金额限制：100U，上不封顶）
4. 接口调用完成后可获取到一个projectId
