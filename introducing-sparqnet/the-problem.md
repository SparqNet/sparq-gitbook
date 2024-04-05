---
description: What problem is SparqNet looking to solve?
---

# The Problem

Smart contracts often lack speed and flexibility when running on blockchains with Virtual Machines (VMs). This is common on Ethereum and other EVM-based chains where everyone shares one generic computer with limited throughput.

This design is inefficient because it forces the chain to be both generic and decentralized. This limits the types of applications you can decentralize and how much you can decentralize them.

For example, in the case of the Ethereum Virtual Machine, you can not do the following:

* Loop a function more than 50 times due to the block gas limit.
* Have a stack size larger than 16 variables due to constraints on the EVM itself.
* Parallelize multiple contract calls. Basically, every time a new block has multiple transactions that interact with multiple different contracts, you have to load the contract, parse and save changes to the database for each single one of these contracts, in order.
