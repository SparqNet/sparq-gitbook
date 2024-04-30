---
description: How an EVM contract calls a C++ contract in AppLayer.
---

## Calling EVM contracts from C++

To invoke EVM contract functions from C++, we leverage a templated approach that mimics the contract's functions in a C++ class. This method provides a type-safe way to interact with contracts written in Solidity or other EVM-compatible languages.

First, we define a proxy C++ class that represents the EVM contract. This class will include stubs of the contract's functions, which do not contain actual logic but serve to match the contract's interface in the blockchain:

```c++
class SolMyContract {
  public:
    uint256_t myFunction(const uint256_t& arg1, const uint256_t& arg2) const {};
    static void registerContract() {
      ContractReflectionInterface::registerContractMethods<SolMyContract>(
        std::vector<std::string>{},  // List of dependencies or related artifacts if any
        std::make_tuple("myFunction", &SolMyContract::myFunction, FunctionTypes::View, std::vector<std::string>{"arg1", "arg2"})
      );
    }
};
```

Then, we ensure that the proxy class is registered within the blockchain before any calls are made. Typically, this registration is done once, often in the constructor of the calling C++ contract, to set up the reflection system used for method invocation:

```c++
uint256_t AnotherContract::callMyFunction(const Address& targetAddr, const uint256_t& arg1, const uint256_t& arg2) const {
  SolMyContract::registerContract();  // Ensure the EVM contract's methods are registered (Can be done only a single time in the constructor)
  return this->callContractViewFunction<SolMyContract>(this, targetAddr, &SolMyContract::myFunction, arg1, arg2);
}
```
