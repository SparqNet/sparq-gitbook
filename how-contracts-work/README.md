---
description: How smart contracts work on the SparqNet protocol.
---

# How contracts work

Contracts in SparqNet are custom, developer-made classes that directly interact with the current State of the Blockchain. Similar to Solidity contracts, they can be used to employ any type of logic within the network, but unlike Solidity, they are coded _natively_ instead of using an intermediary language. This means they aren’t subject to EVM constraints, so we can take advantage of that and have full control of the contract's logic, unleashing blazing fast performance, flexibility and power.

This chapter will comprehensively cover creating new contracts for SparqNet using OrbiterSDK. In general terms, to create a contract from scratch, you have to create the contract logic in C++ and manually code several transaction parsing methods to parse the arguments of a given transaction calling your contract. Additionally, you have to concern yourself with storing your local variables in a database. These are some of the reasons SparqNet is creating a Solidity to C++ transpiler.

The rules explained throughout this chapter ensure that contracts remain compatible with existing frontend Web3 tools (e.g. MetaMask, ethers.js, web3.js, etc.). Those are designed to interact with Solidity contracts and thus require a similar interface.

To call your contract's functions from a frontend, you'll also need to generate its ABI - you can either do it directly with our generator tool (explained further), or replicate their definitions in Solidity and use an external tool like Ethereum's [Remix](https://remix.ethereum.org/) or any other of your preference. This ABI can then be used by your preferred Web3 frontend.
