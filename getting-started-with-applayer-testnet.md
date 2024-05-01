---
description: A hands-on guide for interacting with the AppLayer Testnet
---

# Getting started with AppLayer Testnet

Here's a simple guide on how to start developing with AppLayer and the Blockchain Development Kit (BDK). We'll be using the [Remix IDE](https://remix.ethereum.org) as a tool to aid us in deploying contracts in the network.

### Step 1 - Switching to the AppLayer Testnet

First, make sure you have set your preferred Web3 frontend (e.g. MetaMask) to connect to the AppLayer Testnet. Configuration is as follows:

* **Network Name**: AppLayer Testnet
* **RPC URL**: https://testnet-api.applayer.com/
* **Chain ID**: 75338
* **Currency symbol**: APPL

Keep in mind that we don't have a block explorer at the moment, but this is expected to change in the future.

Here's a video showing the configuration on MetaMask as an example:

{% file src=".gitbook/assets/applayer_step1.mp4" %}

### Step 2 - Claiming your AppLayer Tokens

To claim your APPL tokens on the testnet, reach out to our staff on Discord (see "Join our Community") and open a ticket there. We're currently working on a faucet site that will make this process more straight-forward in the future.

### Step 3 - Deploying C++ contracts

To deploy a C++ contract on the testnet, open Remix IDE and then compile this Solidity interface in it:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface ContractManager {
  struct Contract {
    string name;
    address addr;
  }

  function getDeployedContracts() external view returns(Contract[] memory);
  function getDeployedContractsForCreator(address creator) external view returns (Contract[] memory);
  function createNewERC20Contract(string calldata name, string calldata ticket, uint8 decimals, uint256 mintValue) external returns(address);
  function createNewNativeWrapperContract(string calldata erc20name, string calldata erc20ticker, uint8 erc20decimals) external returns(address);
  function createNewDEXV2PairContract() external returns(address);
  function createNewDEXV2FactoryContract(address feeToSetter) external returns(address);
  function createNewDEXV2Router02Contract(address factory, address nativeWrapper) external returns(address);
  function createNewERC721Contract(string calldata erc721name, string calldata erc721symbol) external returns(address);
}
```

This will allow you to call the `ContractManager` precompiled contract and use it to deploy any of the available precompiles on the blockchain. Precompile deploys cost 100,000 gas each.

To find out the address of your deployed contract, call the `getDeployedContractsForCreator()` function, passing the `ContractManager` address itself as the argument. See the example video that deploys an ERC20 contract:

{% file src=".gitbook/assets/applayer_step3.mp4" %}

### Step 4 - Deploying EVM contracts

Deploying a Solidity/EVM contract on the testnet is done just like with Ethereum. The AppLayer EVM is set to "Shanghai", so be sure to set Remix to the same version before compiling. Check the video below for an example on the following contract:

```solidity
// contracts/GLDToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract AnotherTestToken is ERC20 {
  constructor(uint256 initialSupply) ERC20("AnotherTestToken", "ATT") {
    _mint(msg.sender, initialSupply);
  }
}
```

{% file src=".gitbook/assets/applayer_step4.mp4" %}
