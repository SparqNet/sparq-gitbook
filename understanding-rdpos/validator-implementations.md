---
description: How to add validators to the network
---

# Validator implementations

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
