# 2 以太坊核心概念及第一个合约

## 2.1 理解以太坊，运行第一个智能合约

```
pragma solidity ^0.8.0;
// Counter是合约状态变量，保存在链上
contract Counter {
    // 状态变量
    uint public counter;

    constructor() {
        counter = 0;
    }

    // count()是合约函数
    function count() public {
        counter = counter + 1;
    }
}
```

Remix IDE: 

https://remix.ethereum.org

https://remix.learnblockchain.cn/
