---
description: How SparqNet uses Random Deterministic Proof of Stake
---

# Understanding rdPoS

This section explains the rdPoS algorithm used by the SparqNet protocol. Below are the sub-sections covered:

* Blockchains overview
* How rdPoS works
* Validator implementations
* Slashing
* rdPoS implementation

### Blockchains overview

One of the biggest problems of blockchain development is handling rollbacks. For example, on the Bitcoin chain, assuming there is a latest block that has another block after it. If a node receives a block that replaces the latest block, the next block and all the transactions in it are replaced too, which results in a rollback of the blockchain’s state by one block.

The Bitcoin blockchain and other derivatives follow the longest lived chain rule (the chain with the most accumulated proof of work is the main chain). However, rollbacks unearth problems in that rule. For instance, when a developer is building dApps where they have to deal with such special conditions, it could take greater effort depending on the size/complexity of the application.&#x20;

#### Avoiding rollbacks and achieving consensus on ultra-fast networks

<figure><img src="../.gitbook/assets/Diagram 6.png" alt=""><figcaption></figcaption></figure>

In the above diagram, block C was replaced by block D followed by block E, rolling back the transactions made in block C. The solution to the problem is avoiding the rollback condition altogether. This can be done by deterministically defining which network node can create a block, thereby eliminating the block race condition and keeping everyone in the network synchronized.

The SparqNet project implements this concept as Random Deterministic Proof of Stake (rdPoS), which pairs a block congestion system and a random generator system, allowing only one validator to create a block in a given time.

### How rdPoS works

Within the SparqNet code is RandomGen (random.h), a deterministic uint256\_t generator used for almost everything related to consensus. This deterministic randomness ensures that every node has a chance to respond to a given request (block, randomness, bridging, etc.), while making sure that the nodes selected from the network are truly random and not problematic nodes selected by a malicious actor.

For RandomGen to be viable, it needs to be seeded with a truly random number. Our solution for that is as follows:

* Every time a new block is about to be created, 16 random nodes are selected using RandomGen with the previous block’s randomness seed.
* These nodes make a 32-byte random string (RandomnessSeed) and hash it (RandomnessHash), then sign the hash and publish it to the network.
* After all the nodes have signed and published their hashes to the network, they can post the true data verifying that no one is trying to manipulate the end result.
* After the data is published and included in the block, the randomness seeds are concatenated and hashed, and the resulting hash is used to seed the next block creation.

We have to pay attention to the current state of RandomGen to ensure that all nodes are always in the same internal state so they can properly synchronize with each other.

#### Practical validator rules in the complete block creation process

1. A list of network validators is randomly generated and sorted using the "randomness" seed from the previous block.
2. The first validator from the list will be the block creator, while at least 4 others will create a random 32-byte string and make two transactions with it: one containing the hash of said string, and another containing the string itself, both signed.
3. The hashes are verified to make sure they match their respective random strings.
4. A new block is created by the first validator, concatenating and hashing the other validators' random strings to create a new "randomness" seed that will be used next time on step 1.
5. The block is signed and published to the network by the first validator, while the other validators verify that all transaction signatures (random and hashed) correspond with the list in step 1.
6. The genesis block enforces a given fixed randomness to be valid, since there is no previous block before genesis to derive the randomness from. Additionally, at least five hardcoded validators are needed to bootstrap the network, since each block requires at least 4 validators to confirm the string and hash transaction signatures, and one for signing the block itself.&#x20;

### Validator implementations

Developers get to choose how validators are added to the network, but there are three pre-established implementation options: decentralized, centralized, and semi-decentralized.

#### Decentralized

In a decentralized implementation, all validators have to participate in block creation to ensure that there’s no collusion. A totally decentralized network using rdPoS may face problems when the number of validators on the network grows massively (reaching 10,000, for example) because the latency between those nodes could increase significantly.

To solve this problem, the block time in a decentralized network should be longer (between 15 and 30 seconds, for example), so all the nodes have enough time to respond. Validators can be added to a decentralized network by locking a certain amount of tokens in the BlockManager contract (the class that has the rdPoS logic).

#### Centralized

In a centralized implementation, every network has a master address that can add as many validators as desired. The developer who sets up this implementation is in charge of keeping the chain up and running. The recommended number of nodes for this implementation is at least 32, but you can use more or less nodes depending on the application’s needs.

#### Semi-decentralized

In a semi-decentralized implementation, both validator types are used:

* A normal validator, simply called validator, similar to the decentralized one and added to the network the same way (by locking tokens).
* A validator called sentinel, similar to the centralized one and added to the network the same way (with a master address).

This implementation is different in that neither validators nor sentinels can create a block on their own. The randomness requires at least one of the transactions from a sentinel and whoever publishes the block has to follow the validator list order.

This means the network can have a smaller number of validators (like 16, for example), requiring less computing power for verification but remaining highly secure. Since sentinels take part in the process, every extra byte in the concatenated randomness seed will change the resulting hash.

### rdPoS implementation

With the current code, the BlockManager class maintains and applies all the rdPoS logic. The "randomness" engine is in utils/random.h and also includes vector sorting. At the moment, there is a prototype of a centralized implementation.

\
