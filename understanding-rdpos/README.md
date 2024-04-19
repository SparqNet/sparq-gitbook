---
description: How SparqNet uses Random Deterministic Proof of Stake.
---

# Understanding rdPoS

To keep consensus on such a blazing fast network without tripping up and/or having to deal with rollbacks, we need a _random deterministic block creation_ that allows only one given node to create a block for a given time, eliminating the risk of a block race condition in the network.

This is implemented in SparqNet as its own Avalanche Subnet-compatible consensus algorithm, called **rdPoS (Random Deterministic Proof of Stake)**, which empowers Validators and Sentinels to deal with block congestion and random number generation.

This chapter aims to explain in-depth the rdPoS algorithm used by the SparqNet protocol, from concept to implementation. With the current code, the BlockManager class maintains and applies all the rdPoS logic. The "randomness" engine (RandomGen) is in `utils/random.h` and also includes vector sorting. At the moment, there is a prototype of a centralized implementation.
