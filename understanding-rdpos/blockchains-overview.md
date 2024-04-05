---
description: How existing blockchains handle rollbacks
---

# Blockchains overview

One of the biggest problems of blockchain development is handling rollbacks. For example, on the Bitcoin chain, assuming there is a latest block that has another block after it. If a node receives a block that replaces the latest block, the next block and all the transactions in it are replaced too, which results in a rollback of the blockchain’s state by one block.

The Bitcoin blockchain and other derivatives follow the longest lived chain rule (the chain with the most accumulated proof of work is the main chain). However, rollbacks unearth problems in that rule. For instance, when a developer is building dApps where they have to deal with such special conditions, it could take greater effort depending on the size/complexity of the application.&#x20;

#### Avoiding rollbacks and achieving consensus on ultra-fast networks

<figure><img src="../.gitbook/assets/Diagram 6.png" alt=""><figcaption></figcaption></figure>

In the above diagram, block C was replaced by block D followed by block E, rolling back the transactions made in block C. The solution to the problem is avoiding the rollback condition altogether. This can be done by deterministically defining which network node can create a block, thereby eliminating the block race condition and keeping everyone in the network synchronized.

The SparqNet project implements this concept as Random Deterministic Proof of Stake (rdPoS), which pairs a block congestion system and a random generator system, allowing only one validator to create a block in a given time.
