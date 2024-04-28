---
description: How existing blockchains handle rollbacks.
---

# Blockchains overview

One of the biggest problems of blockchain development is handling block rollbacks. For example, on the Bitcoin chain, assuming there is a latest block that has another block after it. If a node receives a block that replaces the latest block, the next block and all the transactions in it are replaced too, which results in a rollback of the blockchain’s state by one block.

The Bitcoin blockchain and other derivatives follow the "longest lived chain" rule (the chain with the most accumulated proof of work is the main chain). However, rollbacks unearth problems in that rule. For instance, when a developer is building dApps where they have to deal with such special conditions, it could take a greater effort depending on the size and/or complexity of the application.

<figure><img src="../.gitbook/assets/Diagram 6.png" alt=""><figcaption><p>Example of block rollback</p></figcaption></figure>

In the diagram above, block C was replaced by block D followed by block E, rolling back the transactions made in block C.

The solution to the problem is avoiding the rollback condition altogether. This can be done by deterministically defining which network node can create a block, thereby eliminating the block race condition and keeping everyone in the network synchronized to the same latest block.

AppLayer implements this concept as **Random Deterministic Proof of Stake (rdPoS)**, which pairs a block congestion system and a random number generator system, allowing only one Validator to create a block at any given time, thus avoiding rollbacks and achieving consensus on ultra-fast networks.
