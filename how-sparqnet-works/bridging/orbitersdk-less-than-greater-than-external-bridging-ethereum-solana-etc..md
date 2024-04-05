---
description: How SparqNet bridges with other blockchains
---

# OrbiterSDK <-> External Bridging (Ethereum, Solana, etc.)

There are multiple edge cases related to external bridging. For example, it's not possible to natively push data into these chains without paying transaction fees, and those external networks are also limited on both processing power and how much signature verification can be done.

Knowing this, at least for now, the current bridging implementation for them is centralized and owned by Sparq Labs Inc. and its most trusted partners to ensure operational safety.

OrbiterSDK <-> External bridge follows lock/release mechanisms.

### How is safety ensured?

Nodes that will read from a given chain are determined using `RandomGen`, the trustless decentralized random number generator developed by Sparq Labs Inc.

We ensure to keep a "fair" selection of nodes, but even then there is the possibility of a 51% attack - in a network with 100 nodes, if a given malicious user controls 50 of them, and all of them get selected for driving a cross-chain request and a block, they could collude and forward any message they wanted. We avoid this by introducing Sentinels to the network to ensure this collusion doesn't happen.

See the following links for more details on [How rdPoS works](https://github.com/SparqNet/sparq-docs/blob/development/Sparq\_en-US/ch1/1-3.md), [How consensus works](https://github.com/SparqNet/sparq-docs/blob/development/Sparq\_en-US/ch1/1-2.md), [RandomGen](https://github.com/SparqNet/sparq-docs/blob/development/Sparq\_en-US/ch2/2-1-1.md#randomgen)
