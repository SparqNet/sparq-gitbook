---
description: Coding the Simple Contract's source file
---

# Simple Contract Source

With the `SimpleContract` header, declarations and (most of the) registering done, now we can proceed to the implementation itself.

### Defining the Contract Constructors and Destructor

Open the source file (`simplecontract.cpp`) and `#include "simplecontract.h"` right at the beginning.

The first thing we'll implement is the destructor and constructors of our contract class. The implementation must follow a certain order of events:

* The base `DynamicContract` constructor must be called and its respective arguments must be passed in order
* Contract SafeVariables must be accessed with `this` (e.g. `this->name`) and initialized accordingly with their values if necessary (e.g. directly from the constructor, or by fetching values from the database)
* Contract SafeVariables must call `commit()` manually to properly set their values to the values assigned before
* `registerContractFunctions()` must be called to properly register the contract's functions and events
* Contract SafeVariables must call `enableRegister()` manually so they can be set to be properly marked as "used" during contract calls (for the commit/revert logic to work)
* If anything happens during construction that would require throwing an exception, said throw should be done _**before**_ calling `enableRegister()` on any contract SafeVariable - enabling registers should be the _last_ thing done by the constructor to avoid heap-use-after-free errors caused by accessing variables that are accessed after a throw happens

Our source file will look something like this:

```cpp
#include "simplecontract.h"

SimpleContract::SimpleContract(
  const std::string& name,
  const uint256_t& number,
  const std::tuple<std::string, uint256_t>& tuple,
  ContractManagerInterface &interface,
  const Address& address,
  const Address& creator,
  const uint64_t& chainId,
  const std::unique_ptr<DB> &db
) : DynamicContract(interface, "SimpleContract", address, creator, chainId, db),
  name_(this), number_(this), tuple_(this)
{
  this->name_ = name;
  this->number_ = number;
  this->tuple_ = tuple;

  this->name_.commit();
  this->number_.commit();
  this->tuple_.commit();

  registerContractFunctions();

  this->name_.enableRegister();
  this->number_.enableRegister();
  this->tuple_.enableRegister();
}

SimpleContract::SimpleContract(
  ContractManagerInterface &interface,
  const Address& address,
  const std::unique_ptr<DB> &db
) : DynamicContract(interface, address, db), name_(this), value_(this), tuple_(this) {
  this->name_ = Utils::bytesToString(db->get(std::string("name_"), this->getDBPrefix()));
  this->number_ = Utils::bytesToUint256(db->get(std::string("number_"), this->getDBPrefix()));
  this->tuple_ = std::make_tuple(
    Utils::bytesToString(db->get(std::string("tuple_name"), this->getDBPrefix())),
    Utils::bytesToUint256(db->get(std::string("tuple_number"), this->getDBPrefix()))
  );

  this->name_.commit();
  this->number_.commit();
  this->tuple_.commit();

  registerContractFunctions();

  this->name_.enableRegister();
  this->number_.enableRegister();
  this->tuple_.enableRegister();
}

SimpleContract::~SimpleContract() {
  this->db_->put(std::string("name_"), Utils::stringToBytes(this->name_.get()), this->getDBPrefix());
  this->db_->put(std::string("number_"), Utils::uint256ToBytes(this->number_.get()), this->getDBPrefix());
  this->db_->put(std::string("tuple_name"), Utils::stringToBytes(get<0>(this->tuple_)), this->getDBPrefix());
  this->db_->put(std::string("tuple_number"), Utils::uint256ToBytes(get<1>(this->tuple_)), this->getDBPrefix());
}
```

Notice that, in the first constructor, we use `SimpleContract` as the `contractName` argument in the base `DynamicContract` constructor. As stated in the previous subchapter, this match is a **requirement**, otherwise it will result in a segfault. Both constructors initialize the inner variables of the contract - the first one using the arguments directly, and the second one loading them directly from the database.

The destructor is responsible for saving the contract variables to the database, so that they can be loaded later by the second constructor, when `ContractManager` is being constructed. `getDBPrefix()` is a getter for the contract's own prefix in the database, which would be equivalent to `DBPrefix::contracts` + the contract's address.

Keep in mind that **the database stores data as raw bytes** - this is why we use the respective conversion functions from Utils when saving (`XyzToBytes()`) and loading (`bytesToXyz()`) variables.

### Defining the Contract Functions

Now, let's implement the proper functions of our contract - first, the **view** functions (that only read and never change the contract's variables when called), then, the **non-view** functions (that _do_ change the contract's variables when called).

**View** functions MUST be `const`, while **non-view** functions MUST NOT be `const`, and both functions can return either `void` or one of the ABI-supported types.

In our case, we have three view functions which would be `getName()`, `getNumber()` and `getTuple()`, which are the getters for the variables of our contract - `name`, `number` and `tuple`, respectively. We can return the inner data from any SafeVariable by calling the `get()` function, like this:

```cpp
std::string SimpleContract::getName() const { return this->name_.get(); }
uint256_t SimpleContract::getNumber() const { return this->number_.get(); }
std::tuple<std::string, uint256_t> SimpleContract::getTuple() const {
  return std::make_tuple(get<0>(this->tuple_), get<1>(this->tuple_));
}
```

For the three non-view functions we have, which would be the setters (`setName()`, `setNumber()` and `setTuple()` respectively), we must also check that whoever is calling those functions is the actual creator of the contract, as we want to prevent unwanted calls from other addresses (this is how it's coded in the original Solidity code reference mentioned at the start of this subchapter). We can do that by calling `getCaller()` and `getContractCreator()`, respectively, to access the address of the caller and the address of the contract creator, and then we check if both addresses are the same.

If your contract has events, you can emit them by simply calling them like they were any other function (which they actually are if you think about it!). The only thing you have to pay attention to is that **events can ONLY be emitted from non-view functions**, due to how const correctness works in C++ (view functions are `const`, so trying to emit an event from one of them will result in a compilation error).

```cpp
void SimpleContract::setName(const std::string& argName) {
  if (this->getCaller() != this->getContractCreator()) {
    throw std::runtime_error("Only contract creator can call this function.");
  }
  this->name_ = argName;
  this->nameChanged(this->name_.get());
}

void SimpleContract::setNumber(uint256_t argNumber) {
  if (this->getCaller() != this->getContractCreator()) {
    throw std::runtime_error("Only contract creator can call this function.");
  }
  this->number_ = argNumber;
  this->numberChanged(this->number_.get());
}

void SimpleContract::setTuple(const std::tuple<std::string, uint256_t>& argTuple) {
  if (this->getCaller() != this->getContractCreator()) {
    throw std::runtime_error("Only contract creator can call this function.");
  }
  this->tuple_ = argTuple;
  this->tupleChanged(std::make_tuple(get<0>(this->tuple_), get<1>(this->tuple_)));
}
```

Notice when we use `get<>()` on the tuple, we're NOT calling `std::get<>()`, but rather the `SafeTuple`'s own `get<>()` implementation. This is due to how SafeVariables work internally - as it is custom functionality, the STD library is unaware of it, thus `std::get<>()` is not going to work (or even compile for that matter).

### Registering the Contract Functions

After all functions are implemented, we must implement one more - `registerContractFunctions()`, which is responsible for registering the other functions so they can be called later by a transaction or an RPC `eth_call`.

The first thing it should do is call `registerContract()` right away, so it's guaranteed that the contract itself will be registered before its functions.

When registering the contract's functions, their respective functors/signatures will be stored in an internal map, allowing a given transaction to call any function within that contract. Registration is done within try/catch blocks internally, which allows the protection of SafeVariables against any exceptions thrown by the function.

As for the functions themselves, they are registered by calling `this->registerMemberFunction()` for each function your contract has (NOT including events), always passing four arguments to it: the function's name, a reference to the function, its state mutability, and `this` (a pointer to the contract itself).

```cpp
void SimpleContract::registerContractFunctions() {
  registerContract();
  this->registerMemberFunction("getName", &SimpleContract::getName, FunctionTypes::View, this);
  this->registerMemberFunction("getNumber", &SimpleContract::getNumber, FunctionTypes::View, this);
  this->registerMemberFunction("getTuple", &SimpleContract::getTuple, FunctionTypes::View, this);
  this->registerMemberFunction("setName", &SimpleContract::setName, FunctionTypes::NonPayable, this);
  this->registerMemberFunction("setNumber", &SimpleContract::setNumber, FunctionTypes::NonPayable, this);
  this->registerMemberFunction("setTuple", &SimpleContract::setTuple, FunctionTypes::NonPayable, this);
}
```

Note that the complete implementation of this contract has overloads on `getNumber()`, so you may see `static_cast<uint256_t(SimpleContract::*)() const>(&SimpleContract::getNumber)` in place of simply `&SimpleContract::getNumber` - it's done this way so we know exactly which function we are referring to. Since this example is only partial and we only have one `getNumber()` function in it, we're referencing it here normally as simply `&SimpleContract::getNumber`.
