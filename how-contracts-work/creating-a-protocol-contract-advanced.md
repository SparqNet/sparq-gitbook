# Creating a Protocol Contract (Advanced)

We highly recommend using [Doxygen](https://doxygen.nl) to generate the documentation of the current source code, so you can better understand the structure of the project and how your contract can fit in.

You can create any contract you want, but there are some requirements that you need to follow:

### Add the Protocol Contract to theprotocolContractAddresses map

Under `src/contract/contractmanager.h`, you will find a global `std::unordered_map` which contains the registered Protocol Contract addresses. You should stick to the same format as the other contracts, and add your contract address to the map.

### Pay attention to the contract's state

It is required to pay attention to the state of the contract and how nodes will interact with each other. For example, if you have a contract that is doing parallel processing of a task, you need to make sure that this task can be replicated with the same result in any node. Not taking care of the state of a contract will eventually lead to undefined behavior, and that's a hornet's nest nobody wants to touch, right?

### Override `ethCall()` and take care of your revert conditions

It is necessary to override the `ethCall()` function in order to make functions callable by a transaction or an RPC `eth_call`. Protocol Contracts don't have automatic function parsing and they don’t handle call reverts, so you need to handle it all yourself.

### Pay attention to `this->getCommit()`

When `this->getCommit() == false`, it means that the current call is trying to simulate if it's going to throw or not, but if `this->getCommit() == true`, it means that the current call is trying to commit to the state, and you should do it respectively if it doesn't throw.

### Add a reference to `ContractManager`

It is necessary to add a reference of your Protocol Contract to the `ContractManager`. This is done within the `rdPoS` contract (another Protocol Contract). See more under these files:

* `src/contract/contractmanager.h`
* `src/contract/contractmanager.cpp`
* `src/core/rdpos.h`
* `src/core/rdpos.cpp`
