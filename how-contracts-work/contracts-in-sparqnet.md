# Contracts in SparqNet

## Contracts in SparqNet

Contracts in Sparq work similar to Solidity contracts - developers can implement diverse logic structures within the network, and directly interact with the blockchain's current state. The main difference is, in Sparq, contracts are _native, compiled C++ code_, taking advantage of the absence of an EVM and its constraints, having full control of the contract's logic and unleashing blazing fast performance.

This chapter will comprehensively cover creating new contracts for Sparq using OrbiterSDK. Generally, to create a contract from scratch, you must:

* Develop the contract's logic in C++
* Manually code several methods to parse arguments of transactions calling your contract
* Use a database to manage the storage of your local variables

The rules explained throughout this chapter ensure that contracts remain compatible with frontend Web3 tools (e.g. MetaMask, ethers.js, web3.js, etc.). Those are designed to interact with Solidity contracts and thus require a similar interface.

To call your contract's functions from a frontend, you can replicate their definitions in Solidity and generate the ABI using our generator tool (explained further), or an external tool like Ethereum's [Remix](https://remix.ethereum.org/) or any other of your preference. This ABI can then be used with your preferred Web3 frontend.

&#x20;

### Types of Contracts

OrbiterSDK offers two types of contracts: **Dynamic Contracts** and **Protocol Contracts**. The differences between them primarily come from how they are created and managed within the SDK.

#### Dynamic Contracts (recommended)

* Can only be handled by the `ContractManager` class (see below), which enables the chain owner to create an unlimited number of Dynamic Contracts
* Can use Safe Variables - an additional layer of protection that allows better control over whether their changes are committed to the state or automatically reverted when necessary (e.g. when a transaction fails)
* Can only be called when a block is being processed
* Are directly loaded into memory and work very similarly to Solidity contracts
* OrbiterSDK provides ready-to-use templates for the following Dynamic Contracts: `ERC20`, `ERC20Wrapper`, and `NativeWrapper`

#### Protocol Contracts

* Are directly integrated into the blockchain, therefore not linked to the `ContractManager` class and not contained by it, which removes some restrictions but adds others (see [3-6](https://github.com/SparqNet/sparq-docs/blob/main/Sparq\_en-US/ch3/3-6.md))
* Cannot use Safe Variables as they only work with Dynamic Contracts - so it's all up to you (for the most part) as to where to place the contract's variables and their commit/revert logic within the source code of the blockchain

#### The ContractManager class

<figure><img src="../.gitbook/assets/ContractManager (1).png" alt=""><figcaption></figcaption></figure>

The `ContractManager` class (declared in `src/contract/contractmanager.h`) is a _Protocol Contract_, responsible for:

* Handling all the logic related to creating and loading Dynamic Contracts registered within the blockchain (which you can get a list of by calling the class' `getContractList()` function - it returns an `(address[], string[])` map with the currently registered contracts)
* Managing global variables for contracts, such as the contract's name, address, owner, and balance
* Calling any function registered within your contract if the functor/signature matches
* Automatically committing/reverting changes made to the account state when necessary (for non-view functions)

The `ContractManager` is the class that holds all the current contract instances in the State, besides being the access point for contracts to access other contracts. It's header should be similar to the following:

```cpp
class ContractManager {
private:
     std::unordered_map<Address,std::unique_ptr<Contract>> _contracts;
public:
        ContractManager(std::unique_ptr<DBService> &dbService);
     std::unique_ptr<Contract>& getContract(Address address);
     const std::unique_ptr<const Contract>& getConstContract(Address address) const;
     void processTransaction(const Tx& transaction)  
}
```

The contract manager will be responsible for deploying the contracts in the chain, loading them from DB when constructing and saving them to DB when deconstructing. The function `processTransaction` would be similar to this

#### Example Contract

Giving the example Solidity contract:

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

Declaration

```cpp
ExampleContract.h
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

Definition

```cpp
ExampleContract.cpp
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

**The Contract base class**

The Contract class, declared in contract/contract.h, is the base class which all contracts derive from. This class holds all the [Solidity global variables](https://docs.soliditylang.org/en/v0.8.17/units-and-global-variables.html), besides variables common among these contracts (such as contract Address). Its header should look similar to the following:

```cpp
// class Contract {
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
