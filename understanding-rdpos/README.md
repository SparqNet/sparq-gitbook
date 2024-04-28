---
description: How AppLayer uses Random Deterministic Proof of Stake.
---

# Understanding rdPoS

To keep consensus on such a blazing fast network without tripping up and/or having to deal with rollbacks, AppLayer uses *random deterministic block creation* that allows only one given node to create a block for a given time, eliminating the risk of a block race condition in the network.

This is implemented in AppLayer as its own consensus algorithm, called **rdPoS (Random Deterministic Proof of Stake)**, which empowers Validators and Sentinels to deal with block congestion and random number generation. This chapter aims to explain in-depth the rdPoS algorithm used by the AppLayer protocol, from concept to implementation.
