---
description: How SparqNet's Solidity to C++ Transpiler works
---

# Solidity to C++ Transpiler

### Solidity to C++ Transpiler

Besides the ability to easily bring already existing Solidity developers into the SparqNet space, a Solidity to C++ transpiler can create the intermediary functions between the transaction in the State of the Blockchain and the Contract itself. Doing this dynamically would be counterproductive to one of our core tenets (performance) and introduce development barriers.

For example, The data field of a transaction of a user calling the function transfer(address to, uint256 value) of a given contract with the arguments `0x7e4aa755550152a522d9578621ea22edab204308 and 840000000000000000000` is going to be: `0xa9059cbb0000000000000000000000007e4aa755550152a522d9578621ea22edab20430800000000000000000000000000000000000000000000002d89577d7d40200000`

Where:

* `0xa9059cbb` is the function functor (`keccak256("transfer(address,uint256)").substr(8)`)
* `0000000000000000000000007e4aa755550152a522d9578621ea22edab204308` is the encoded address
* `00000000000000000000000000000000000000000000002d89577d7d40200000` is the encoded uint256

The possibility here is to simply have a `contractManager.processTransaction(tx)` inside the `State::processNewBlock` function, where all transactions that call contracts can be routed through a single place.

However, between `contractManager.processTransaction(tx)` and `transfer(address to, uint256 value)` of said Contract, the arguments need to be parsed. Besides the right function being called, the transpiler translates the solidity source code and the functions needed for argument parsing and function selection.

Aside from the argument and function parsing issues that arise when the contract is not running in a VM, the developer has to consider their local variables and store them in a DB when opening/closing the node.

The Solidity to C++ Transpiler handles these local variables inside the contract. One major difference between Solidity EVM and C++ SparqNet contracts is that by default, databases are only used when opening the node (loading a past state when starting node) and closing the node (saving the current state when closing node).

Local variables are kept in the memory, while with Solidity, every call to a local variable is a database call. If the developer Contract needs to load something from the DB during execution, they are free to do so, but transpiled source code will always be at the constructor/destructor of the Contract.

One of the main features of Solidity is direct contract interaction. You can cast an address to a contract and call it. If said address contains a valid contract that matches the Interface specified by the developer, that contract function is successfully called and returned.

For example:

```cpp
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;
interface IERC20 {
    function balanceOf(address addr) external view returns (uint256);
} 
contract GetERC20Balance {
    function getERC20balance(address contractAddr, address user) external view returns (uint256) {
        IERC20 tokenContract = IERC20(contractAddr);
        return tokenContract.balanceOf(user);
    }
}
```

In the above contract, the EVM casts the `contractAddress` into a contract with the IERC20 interface, enabling the usage of the `balanceOf()` function. On SparqNet, as contracts are compiled directly with the blockchain itself, a class called `ContractManager` (declared in `contracts/contractmanager.h`) can be used to hold all contract classes’ instances in a polymorphic manner.&#x20;

Additionally, a reference from it can be argued to any contract (default in the base `Contract` class). By using polymorphism, you can cast pointers from a given type (a Generic Contract stored inside `unordered_map<Address,Contract>` on `ContractManager`) to a desired contract. This is exemplified in the ContractManager topic.

The equivalent definition in C++ would be similar to:

```cpp
uint256_t GetERC20Balance::getERC20Balance(const Address& address, const Address &user) {
    auto tokenContract = dynamic_cast<const ERC20&>(this->contractManager.getContract(address));
    return tokenContract.balanceOf(address);
}
```

Remember that `getERC20Balance()` is declared as an external view function in Solidity, forcing the contract cast to be `const` only, as this contract function cannot change the state.
