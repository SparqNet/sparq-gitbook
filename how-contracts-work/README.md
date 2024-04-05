---
description: How smart contracts work on the SparqNet protocol
---

# How contracts work

SparqNet Contracts are custom, developer-made classes that directly interact with the current State of the Blockchain. Similar to Solidity contracts, they can be used to employ any type of logic within the network, but unlike Solidity, they aren’t subject to EVM constraints.

To create a new contract on SparqNet, you have to create the contract logic in C++ and code several transaction parsing methods to parse the arguments of a given transaction calling your contract. Additionally, you have to concern yourself with storing your local variables in a database. These are some of the reasons SparqNet is creating a Solidity to C++ transpiler.
