---
description: How a C++ contract calls an EVM contract in AppLayer.
---

## Calling C++ contracts from EVM

Calling a C++ contract from an EVM contract uses a standard Solidity interface to abstract the C++ implementation. This approach ensures that calls from EVM to C++ are as straightforward as EVM-to-EVM calls.

First, we define a Solidity interface that matches the signature of the C++ functions you wish to call. This interface acts as a facade, providing a Solidity view of the C++ contract functionalities:

```solidity
interface MyContract {
    function myFunction(uint256 arg1, uint256 arg2) external view returns (uint256);
}
```

Then, we use the defined interface to make calls to the C++ contract. This is handled similarly to any inter-contract communication in Solidity, ensuring a seamless integration layer:

```solidity
contract AnotherContract {
    function callMyFunction(address cppAddr, uint256 arg1, uint256 arg2) public view returns (uint256) {
        return MyContract(cppAddr).myFunction(arg1, arg2);
    }
}
```
