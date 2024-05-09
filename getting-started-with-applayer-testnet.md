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

To claim your APPL tokens on the testnet, open our [testnet faucet website](https://testnet-faucet.applayer.com/), enter your testnet address there and click on the "Request Tokens" button.

**Please note that MetaMask might take up to 5 minutes to update your balance.** If the faucet displays "Tokens requested and transferred successfully", the operation has been successful, so please be patient.

Check this example video showing how to claim APPL tokens using a MetaMask address:

{% file src=".gitbook/assets/applayer_step2.mp4" %}

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

### Step 5 (Optional) - Using randomness in EVM contracts

You can also use one of our on-chain precompiles called `BDKPrecompile` to fetch random numbers generated on the fly. It is only accessed by the EVM through the following Solidity interface:

```solidity
interface BDKPrecompile {
  function getRandom() external view returns (uint256);
}
```

The precompile is located at the address `0x1000000000000000000000000000100000000001`. To use the interface in your own contracts, you MUST specify this exact address in your contract's code when accessing it, like this:

```solidity
contract MyContract {
  function myFunction() public {
    // ...
    uint256 myRandomNumber = BDKPrecompile(0x1000000000000000000000000000100000000001).getRandom();
    // ...
  }
}
```

As an example, have a look at this Solidity code that represents one of our template contracts used for testing, called `RandomnessTest`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface BDKPrecompile {
  function getRandom() external view returns (uint256);
}

contract RandomnessTest {
  uint256 private randomValue_;
  function setRandom() external {
    randomValue_ = BDKPrecompile(0x1000000000000000000000000000100000000001).getRandom();
  }
  function getRandom() view external returns (uint256) {
    return randomValue_;
  }
}
```

Compile this code in Remix IDE (with the EVM set to "Shanghai", like done in the previous step) and deploy the `RandomnessTest` contract. Once it is deployed call the `setRandom` function to initialize it, and then call the `getRandom` function to get a random number.

**If you are using the interface in a view function, be aware that two different executions will always result in a different value (if called by RPC).** This is done on purpose, as the random value is only decided when the transaction is included in a block and it is generated in a cryptographically secure manner, making it impossible to predict the value. For the `RandomnessTest` contract specifically, due to how it is coded, if you want a new random number you must call `setRandom` again before calling `getRandom` the next time.

See the following video as an example:

{% file src=".gitbook/assets/applayer_step5.mp4" %}
