---
description: Which kinds of pre-compiled contracts can be coded and managed with the BDK.
---

# Dynamic and Protocol Contracts

AppLayer's BDK offers two main classes of contracts: **Dynamic Contracts** and **Protocol Contracts**. The differences between both classes come from how they are created and managed within the BDK.

## Dynamic Contracts (recommended)

* Are managed by the state and can only be created by `ContractManager`, which enables the chain owner to create an unlimited number of Dynamic Contracts
* Can use special types called **SafeVariables** (explained further) - an additional layer of protection that allows better control over whether variable changes are committed to the state or automatically reverted when necessary (e.g. when a transaction fails)
* Can only be called when a block is being processed
* Are directly loaded into memory and work very similarly to Solidity contracts

AppLayer's BDK provides ready-to-use templates for the following Dynamic Contracts:

* `ERC20` (template for an ERC20 token)
* `ERC20Wrapper` (template for an ERC20 wrapper)
* `ERC721` (template for an ERC721 token)
* `NativeWrapper` (template for a native asset wrapper)
* `DEXV2Factory` (template for a DEX factory)
* `DEXV2Library` (namespace for commonly used DEX functions)
* `DEXV2Pair` (template for a DEX contract pair)
* `DEXV2Router02` (template for a DEX contract router)
* `UQ112x112` (namespace for dealing with fixed point fractions in DEX contracts)

There are also specific contracts that only exist for internal testing purposes and are not meant to be used as templates:

* `SimpleContract` (template for a simple contract, used for both testing and teaching purposes)
* `ERC721Test` (derivative contract meant to test the capabilities of the ERC721 template)
* `TestThrowVars` (contract meant to test SafeVariable commit/revert functionality using exception throwing)
* `ThrowTestA/B/C` (contracts meant to test nested call revert functionality)

## Protocol Contracts

* Are directly integrated into the blockchain, therefore not linked to the `ContractManager` class and not contained by the state, which removes some restrictions but adds others (explained further)
* Can be designed to process information beyond transaction calls, communicate with other nodes, access files within the current system, and even automatically call themselves when certain conditions are met - anything is possible, as long as you don't break your own blockchain (more on that later)
* Cannot use SafeVariables nor emit events, as they only work with Dynamic Contracts - so it's all up to the developer (for the most part) as to where to place the contract's variables and their commit/revert logic within the blockchain's source code

## Managing contracts

Contracts in the AppLayer network are managed by a few classes working together (check the `src/core` and `src/contract` folders for more info on each class):

* `State`  (`src/core/state.h`) is responsible for owning all the Dynamic Contracts registered in the blockchain (which you can get a list of by calling the class' `getCppContracts()` and/or `getEVMContracts()` functions, depending on which ones you need), as well as their global variables (name, address, owner, balances, events, etc.)
* `ContractManager` (`src/contract/contractmanager.h`) is responsible for solely creating and registering the contracts, then passing them to State (with the `ContractFactory` namespace providing helper functions to do so)
* `ContractHost` (`src/contract/contracthost.h`) is responsible for allowing contracts to interact with each other and to enable them to modify balances - this kind of inter-communication is done by intercepting calls of functions from registered contracts (if the functor/signature matches), which is done through either an `eth_call` request or a transaction processed from a block
* `ContractStack` (`src/contract/contractstack.h`) is responsible for managing alterations of data, like contract variables and account balances during nested contract call chains, by keeping a stack of changes made during a contract call, and automatically committing or reverting them in the account state when required (for non-view functions)

The `ContractManager` class is itself a _Protocol Contract_, but it does not own or create any Protocol Contracts. Instead, they are created during blockchain initialization, and a reference to each of them is stored within the class, allowing it to access them directly. Dynamic Contracts, however, are fully owned and stored by State in an internal map. This ensures that each contract has a unique address, which is derived using a similar scheme as an EVM.

Because of this, we don't necessarily need to know the type of the contract stored within the pointer, only during either creating it or loading it from the database. This is why all contracts inherit the `BaseContract` class, as it contains a `name_` variable specifying the name of the contract. This name must be the same as the name of the class, as it is used to identify the contract during loading and creation.

Using `ContractHost` for contract inter-communication instead of accessing State directly allows us to do it in an isolated and secure way. For example, if we want to send tokens from a contract to another address, we don't access the balance directly from the State. Every time a function is called by a **user** (not another contract), the balances map available for contracts is empty, only populating what is currently being accessed and not previously available. When modifying the balance, only the mapping within `ContractStack` is modified, therefore allowing for multiple nested contract functions to revert in an atomic fashion while not affecting either the State or other contracts altogether.

## Example of a contract

Given the example Solidity contract:

```cpp
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;
 
contract ExampleContract {
    mapping(address => uint256) values;c
    function setValue(address addr, uint256 value) external {
        values[addr] = value;
        return;
    }
}
```

The transpiled code should look similar to this:

* Declaration (`ExampleContract.h`)

```cpp
#include <...>
class ExampleContract : public DynamicContract {
  private:
    std::unordered_map<Address, uint256_t> values;
    // Const-reference as they are not changed by the function.
    void setValue(const Address &addr, const uint256 &value);
  public:
    ExampleContract(
      const Address& contractAddress, const uint64_t& chainId,
      std::unique_ptr<ContractManager> &contractManager,
      std::unique_ptr<DBService&> db
    );
    void callContractWithTransaction(const Tx& transaction);
}
```

* Definition (`ExampleContract.cpp`)

```cpp
#include "ExampleContract.h"
 
ExampleContract(
  const Address& contractAddress, const uint64_t& chainId,
  std::unique_ptr<ContractManager> &contractManager,
  std::unique_ptr<DBService&> db
) : Dynamic Contract(contractAddress, chainId, contractManager, db) {
  // Read the "values" variables from DB
  // Code generated by the transpiller from all local variables
  // of the solidity contract, on the ExampleContract, you have values as a address => uint256 mapping
  ...
}

void ExampleContract::setValue(const Address &addr, const uint256 &value) {
  this->values[addr] = value;
  return;
}

void ExampleContract::callContractWithTransaction(const Tx& transaction) {
  // CODE GENERATED BY THE TRANSPILLER
  // USED TO ROUTE AND DECODE TRANSACTIONS
  // THE IF USED HERE IS FOR EXAMPLE PURPOSES
  // THE GENERATED CODE WILL BE USING DIFFERENT STRING ALGORITHMS IN ORDER TO MATCH
  // FUNCTOR AND ARGUMENTS TO CONTRACT FUNCTION.
  std::string_view txData = transaction.getData();
  auto functor = txData.substr(0,8);
  // Keccak256("setValue(address,uint256)")
  if (functor == Utils::hexToBytes("0x48461b56")) {
    this->setValue(ABI::Decoder::decodeAddress(txData, 8), ABI::Decoder::decodeUint256(txData, 8 + 32));
  }
  return;
}              
```

## The BaseContract class

The `BaseContract` class, declared in `src/contract/contract.h`, is the base class which all contracts derive from. This class holds all the [Solidity global variables](https://docs.soliditylang.org/en/v0.8.17/units-and-global-variables.html), besides variables common among these contracts (such as contract address). Its header should look similar to the following:

```cpp
class BaseContract {
  private:
     // CONTRACT VARIABLES
     const Address _contractAddress;
     const uint64_t _chainId;
     const std::unique_ptr<ContractManager>& _contractManager;
     // GLOBAL VARIABLES
     static Address _coinbase; // Current Miner Address
     static uint256_t _blockNumber; // Current Block Number
     static uint256_t _blockTimestamp; // Current Block Timestamp
  public:
     Contract(const Address& contractAddress, const uint64_t& chainId, std::unique_ptr<ContractManager> &contractManager) : _contractAddress(contractAddress), _chainId(chainId), _contractManager(contractManager) {}
     const Address& coinbase() { return _coinbase };
     const uint256_t& blockNumber() { return _blockNumber};
     const uint256_t blockTimestamp() { return _blockTimestamp};
     virtual void callContractWithTransaction(const Tx& transaction);
     virtual std::string ethCallContract(const std::string& calldata) const;
     friend State; // State can update the private global variables of the contracts
}
```

Regarding the `callContractWithTransaction` and the `ethCallContract` functions, `callContractWithTransaction` is used by the State when calling from `processNewBlock()`, while `ethCallContract` is used by RPC to answer for `eth_call`. Strings returned by `ethCallContract` are hex strings encoded with the desired function result.
