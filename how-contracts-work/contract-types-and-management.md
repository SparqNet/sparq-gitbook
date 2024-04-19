---
description: >-
  Which kinds of contracts can be coded with OrbiterSDK (and how thy are
  managed).
---

# Contract types and management

OrbiterSDK offers two main types of contracts: **Dynamic Contracts** and **Protocol Contracts**. The differences between both types come from how they are created and managed within the SDK.

#### Dynamic Contracts (recommended)

* Can only be handled by the `ContractManager` class (explained further), which enables the chain owner to create an unlimited number of Dynamic Contracts
* Can use special types called **Safe Variables** (explained further) - an additional layer of protection that allows better control over whether variable changes are committed to the state or automatically reverted when necessary (e.g. when a transaction fails)
* Can only be called when a block is being processed
* Are directly loaded into memory and work very similarly to Solidity contracts

OrbiterSDK provides ready-to-use templates for the following Dynamic Contracts:

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

#### Protocol Contracts

* Are directly integrated into the blockchain, therefore not linked to the `ContractManager` class and not contained by it, which removes some restrictions but adds others (explained further)
* Can be designed to process information beyond transaction calls, communicate with other nodes, access files within the current system, and even automatically call themselves when certain conditions are met - anytihng is possible, as long as you don't break your own blockchain (more on that later)
* Cannot use Safe Variables _or_ emit events, as they only work with Dynamic Contracts - so it's all up to you (for the most part) as to where to place the contract's variables and their commit/revert logic within the blockchain's source code

#### Managing contracts

Contracts in OrbiterSDK are managed by a few classes working together:

* `ContractManager` is responsible for:
  * Handling all the ownership and logic related to creating and loading Dynamic Contracts registered within the blockchain (which you can get a list of by calling the class' `getContractList()` function - it returns an `(address[], string[])` map with the currently registered contracts)
  * Managing global variables for contracts, such as the contract's name, address, owner, and balance, as well as the emission and management of their events (if they have any)
  * Calling any function registered within your contract if the functor/signature matches
  * Automatically committing/reverting changes made to the account state when necessary (for non-view functions)
* `ContractManagerInterface`is responsible for allowing contracts to interact with each other and to enable them to modify balances - this kind of inter-communication is done by intercepting calls of functions from registered contracts (if the functor/signature matches), which is done using either one of the following ways:
  * Taking an `eth_call` request as input
  * Taking a transaction as input (for State when processing blocks)
  * Taking a fully constructed `ethCallInfo` as input to simulate a transaction (for RPC to answer `eth_estimateGas` to let the user know if the transaction is going to revert or not)
* `ContractFactory` is responsible for handling contract creation and registration
* `ContractCallLogger` is responsible for managing alterations of data like contract variables and account balances during nested contract call chains, automatically committing or reverting changes made to the account state when necessary

#### The ContractManager class

<figure><img src="../.gitbook/assets/ContractManager (1) (1).png" alt=""><figcaption><p>Source files for ContractManager</p></figcaption></figure>

The `ContractManager` class (declared in `src/contract/contractmanager.h`) is itself a _Protocol Contract_, but it does not own or create any Protocol Contracts. Instead, they are created during blockchain initialization, and a reference to each of them is stored within the class, allowing it to access them directly.

Dynamic Contracts, however, are fully owned and stored in a `std::unordered_map<Address,std::unique_ptr<DynamicContract>>`. This ensures that each contract has a unique address, which is derived using a similar scheme as an EVM. Like said before, you can get a list of currently registered contracts by calling `getContracts()`, which returns a list of the respective contract names and their addresses.

Because of this, `ContractManager` doesn't necessarily know the type of the contract stored within the pointer - it only needs to know the contract during either creating it or loading it from the database. This is why all contracts inherit the `BaseContract` class, as it contains a `name_` variable specifying the name of the contract. This name must be the same as the name of the class, as it is used to identify the contract during loading and creation.

The reason for using `ContractManagerInterface` for contract inter-communication instead of accessing`ContractManager` or the state directly is to be able to do it in an isolated and secure way. For example, if we want to send tokens from a contract to another address, we don't access the balance directly from the state. Every time a function is called by a **user** (not another contract), the balances map available for contracts is empty, only populating what is currently being accessed and not previously available. When modifying the balance, only the mapping within `ContractCallLogger` is modified, therefore allowing for multiple nested contract functions to revert in an atomic fashon while not affecting either the state or other contracts altogether.

#### Example of a contract

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
class ExampleContract : public Contract {
    private:
        std::unordered_map<Address, uint256_t> values;
        // Const-reference as they are not changed by the function.
        void setValue(const Address &addr, const uint256 &value);
    public:
        ExampleContract(const Address& contractAddress,
                        const uint64_t& chainId,
                        std::unique_ptr<ContractManager> &contractManager, std::unique_ptr<DBService&> db);
        void callContractWithTransaction(const Tx& transaction)
}
```

* Definition (`ExampleContract.cpp`)

```cpp
#include "ExampleContract.h"
 
ExampleContract(const Address& contractAddress,
                const uint64_t& chainId,
                std::unique_ptr<ContractManager> &contractManager, std::unique_ptr<DBService&> db) :
                Contract(contractAddress, chainId, contractManager) {
                    // Read the "values" variables from DB
                    // Code generated by the transpiller from all local variables
                    // of the solidity contract, on the ExampleContract, you have values as a address => uint256 mapping
                    ...
                }
void ExampleContract::setValue(const Address &addr, const uint256 &value) {
    this->values[addr] = value;
    return
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

**The BaseContract class**

The `BaseContract` class, declared in `src/contract/contract.h`, is the base class which all contracts derive from. This class holds all the [Solidity global variables](https://docs.soliditylang.org/en/v0.8.17/units-and-global-variables.html), besides variables common among these contracts (such as contract Address). Its header should look similar to the following:

```cpp
// class BaseContract {
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
