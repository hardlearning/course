# Foundry Fundamentals Section 1: Foundry Simple Storage

## Foundry Install

官方文档：https://getfoundry.sh/

```
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

给`~/.bash_profile`添加环境变量：

```
export PATH="$PATH:/home/abc/.foundry/bin"
```

检查是否安装成功：

```
forge --version
cast --version
anvil --version
chisel --version
```

## Foundry Setup

```
# 在当前目录初始化项目
forge init
```

## Formatting Solidity in VSCode

安装VSCode插件：Solidity - Nomic Foundation 和 Even Better TOML，并添加如下配置：

```
"[solidity]": {
    "editor.defaultFormatter": "NomicFoundation.hardhat-solidity"
}
```

## Compiling in Foundry

```
forge compile
```

## Deploying to a local chain I (Anvil or Ganache)

```
anvil
```

配置MetaMask网络：

- Network Name: Localhost
- Default RPC URL: http://127.0.0.1:8545
- Chain ID: 31337
- Currency symbol: ETH

## Deploying to a local chain II (Forge Create)

```
forge create SimpleStorage --rpc-url http://127.0.0.1:8545 --interactive
forge create SimpleStorage --rpc-url http://127.0.0.1:8545 --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
```

## Deploying to a local chain III (Forge Script)

- script/DeploySimpleStorage.s.sol

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

import {Script} from "forge-std/Script.sol";
import {SimpleStorage} from "../src/SimpleStorage.sol";

contract DeploySimpleStorage is Script {
    function run() external returns (SimpleStorage) {
        vm.startBroadcast();
        SimpleStorage simpleStorage = new SimpleStorage();
        vm.stopBroadcast();
        return simpleStorage;
    }
}
```

```
forge script script/DeploySimpleStorage.s.sol --rpc-url http://127.0.0.1:8545 --broadcast --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
```

## What is a transaction (But actually)

```
cast --to-base 0x714c2 dec
```

## Private Key Rant II

- .env

```
PRIVATE_KEY=0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
RPC_URL=http://127.0.0.1:8545
```

```
source .env
echo $PRIVATE_KEY
echo $RPC_URL
forge script script/DeploySimpleStorage.s.sol --rpc-url $RPC_URL --broadcast --private-key $PRIVATE_KEY
```

```
cast wallet import defaultKey --interactive
cast wallet list
cat ~/.foundry/keystores/defaultKey
forge script script/DeployFundMe.s.sol:DeployFundMe --rpc-url http://localhost:8545 --account defaultKey --sender 0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266 --broadcast -vvvv
```

## Cast Send

```
cast send 0xcf7ed3acca5a467e9e704c703e8d87f634fb0fc9 "store(uint256)" 123 --rpc-url $RPC_URL --private-key $PRIVATE_KEY
cast call 0xcf7ed3acca5a467e9e704c703e8d87f634fb0fc9 "retrieve()"
cast --to-base 0x000000000000000000000000000000000000000000000000000000000000007b dec
```

## Installing foundry-zksync

```
git clone https://github.com/matter-labs/foundry-zksync.git
cd foundry-zksync
./install-foundry-zksync
foundryup-zksync
# switch to vanilla foundry
foundryup
```

## foundry-zksync Compiling

```
forge build --zksync
```