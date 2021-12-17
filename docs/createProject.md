---
id: createProject
title: 创建项目
sidebar_label: 创建项目
---

**创建Project需要调用的接口**

```solidity
function regist(
        string memory name,
        address oper,
        uint256 depositAmt
) returns (uint256 projectId)
```

**相关事件**
```
NewProject(uint256 projectId, string name, address oper, uint256 depositAmt);
```

# 创建Project流程

1. DAPP开发商事先存入足够的USDT代币到调用register接口的账户
2. 调用ERC20的approve方法授权足够的金额给zkRandom合约
3. 调用zkRandom合约的regist方法，第一个参数填项目名称、第二个参数填DAPP合约地址、第三个参数填质押金额（该金额有一个最小金额限制：100U，上不封顶）
4. 接口调用完成后可获取到一个projectId
