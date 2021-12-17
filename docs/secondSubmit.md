---
id: secondSubmit
title: 链下服务器触发二次提交
sidebar_label: 链下服务器触发二次提交
---

在用户调用Dapp合约的预获取随机数方法时，zkRandom会发出NewMessage事件
```
NewMessage(uint256 itemId, bytes32 requestKey, bytes32 hv, address callback, address origin);
```

Dapp链下服务器需要监听此事件，监听到此事件后调用Dapp合约的二次提交方法，由Dapp合约路由到zkRandom合约获取最终的确认的随机数。

伪代码如下:
```
async commit(requestKey) {
  // 获取item 对应的私钥
  const  item_privatekey = ...;
  //获取zkrandom合约对象
  const  ZKRandomContract = ....;
  //获取 dapp 合约对象
  const  DAPPcontract = ....;
  // 网络对应的chainId 
  const  chainid = ....;
  // 数据提交有效时间   当前区块时间 + 失效时间
  const  deadline = nowblockTime  +  timeout;
  let data = await ZKRandomContract.AcceptMessages(requestKey);
  const abiCoder = new ethers.utils.AbiCoder();
  const bytesData = abiCoder.encode(['bytes32','bytes32','uint256','uint256','bytes32','address','uint256','bool','uint256','uint256'] ,
  [
      "0x389b1809e53aa51d6484fccfd505daadbc37caad08990e701622c807662436a0" ,
        data.requestKey , 
        data.itemId,
        data.nonce,
        data.hv,
        data.callback,
        data.created,
        data.isDead,
        deadline,
        chainid]);
  let  itemUser =  new ethers.Wallet(item_privatekey);
  const  sign = await itemUser.signMessage( ethers.utils.arrayify(ethers.utils.keccak256(bytesData)))
  const s1 = ethers.utils.splitSignature(sign);
  const  tx_hash = await DAPPcontract.generateRandom(requestKey,deadline,s1.v,s1.r,s1.s);
  return tx_hash;
}
```
