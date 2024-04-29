---
description: A primer on EVM smart contracts in the AppLayer ecosystem.
---

# EVM contracts

Aside from native contracts, AppLayer can also execute Solidity contracts as-is by the use of the AppLayer EVM, which is compatible with bytecode deployment. This means any language that compiles to EVM bytecode (e.g. Solidity, Viper, etc.) can be used to deploy contracts in the AppLayer EVM in a seamless, straight-forward way.

This kind of compatibility is possible thanks to the integration of the [EVMOne](https://github.com/ethereum/evmone) virtual machine (originally made by the Ethereum developers) and EVMC libraries.
