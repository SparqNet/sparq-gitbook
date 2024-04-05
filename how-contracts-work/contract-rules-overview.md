# Contract rules overview

When creating contracts for Sparq, there are a few rules you must follow to ensure they work as intended. While each contract type has its own rules, some other rules apply to both. This will be explained and demonstrated further in the next few subchapters.

### General contract rules

As a general stance, contracts must:

* Inherit from their respective base class, depending on their type (see below) and make sure you're passing the right arguments for their constructors
* Manage variables within the state and database during contract construction (loading) _and_ destruction (saving)
* Register callbacks for contract functions with their proper functors/signatures (if functions are called by an RPC `eth_call` or a transaction)
* Ensure that their assigned name and their own class name match
  * Both contract constructors take a `contractName` string as an argument - i.e. if your contract is called "TestContract", your constructor's definition would be `TestContract(...) : DynamicContract(interface, "TestContract", ...)` - _both HAVE to match_, otherwise a segfault may happen
* Declare view functions (functions that do not change state) as `const` and return a `std::string` with the encoded ABI (e.g. `std::string getBalance(Address add) const { return ABI::Encoder({balance}).getRaw(); }`)
* Declare callable functions (functions that _do_ change state and are callable by transactions) as non-`const` and return `void` (e.g. `void transfer(Address to, uint256_t value) { ... }`)

### Rules for Protocol Contracts

Protocol Contracts specifically must:

* Inherit `BaseContract` from `src/contract/contract.h`:

<figure><img src="../.gitbook/assets/BaseContract.png" alt=""><figcaption></figcaption></figure>

* Override `ethCall()` functions to parse transaction arguments, manage state changes during their processing (depending on whether the call is committing or not), and commit/revert variables when necessary

### Rules for Dynamic Contracts

Dynamic Contracts specifically must:

* Inherit `DynamicContract` from `src/contract/dynamiccontract.h`:

<figure><img src="../.gitbook/assets/DynamicContract.png" alt=""><figcaption></figcaption></figure>

* Override `registerContractFunctions()` and call it inside the contract's constructor
* Provide two constructors: one for creating the contract from scratch within `ContractManager`, and one for loading the contract from the database
* Only allow contract creation through a transaction call to the `ContractManager` contract
* Develop functions for handling your contract's creation and logic
* Override `ethCall()` functions to register and properly call those functions
* Set **all** of the contract's internal variables as `private`, inherit them from one of the many Safe Variable classes, and always reference them with `this` to ensure correct semantics
  * e.g. `string name` and `uint256 value` in Solidity should be `SafeString name` and `SafeUint256_t value` in C++, respectively - referencing them in your definition would be `this->name`, `this->value`...
* Allow loops using containers such as `SafeUnorderedMap`, but keep in mind how Safe Containers work
  * e.g. when you access a key from a `SafeUnorderedMap`, it'll check if it exists and copy _only_ the key, not the entire map or its value - thus when iterating a loop, you can't assume the "temporary" value is the original one
  * We recommended you only loop inside _view_ functions to ensure value safety, but you can do it on non-view functions as well, just be careful when doing so
* Trigger state changes only via transaction calls to contract functions
* Call `updateState(true)` at the end of the contract's constructor

#### Global Contract Variables

For both contract types (Protocol _and_ Dynamic), you can use the following global functions during an `ethCall()`:

| Global Function      | Description                                | return type |
| -------------------- | ------------------------------------------ | ----------- |
| getContractAddress() | Returns the contract's address             | Address     |
| getContractOwner()   | Returns the contract's owner               | Address     |
| getContractChainId() | Returns the contract's chainId             | uint64\_t   |
| getContractName()    | Returns the contract's name                | string      |
| getOrigin()          | Returns the transaction's origin           | Address     |
| getCaller()          | Returns the transaction's caller           | Address     |
| getValue()           | Returns the transaction's value            | uint256\_t  |
| getCommit()          | Returns if the call is committing to state | bool        |

For Dynamic Contracts specifically, you can also use the following global functions:

| Global Function                       | Description                           | return type |
| ------------------------------------- | ------------------------------------- | ----------- |
| const getContract(address)            | Returns a contract of type T          | const T     |
| callContract(address, ABI, callValue) | Calls contract function               | void        |
| getBalance(address)                   | Get the current balance of an address | uint256\_t  |
| sendTokens(address, value)            | Send tokens to an address             | void        |
