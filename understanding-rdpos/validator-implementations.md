---
description: How to add Validators to the network.
---

# Validator implementations

Developers get to choose how Validators are added to the network, but there are three pre-established implementation options: *permissionless*, *permissioned*, and *semi-permissioned*.

## Permissionless

In a permissionless implementation, all Validators have to participate in block creation to ensure that there’s no collusion. A totally permissionless network using rdPoS may face problems when the number of Validators in the network grows massively (e.g. 10,000 Validators), because the latency between those nodes could increase significantly.

To solve this problem, the block time in a permissionless network should be bigger (e.g. 15-30 seconds) so all the nodes have enough time to respond. Validators can be added to a permissionless network by locking a certain amount of tokens in the contract that contains the rdPoS logic.

## Permissioned

In a permissioned implementation, every network has a "master address" that can add as many Validators as desired. The developer who sets up this implementation is responsible for keeping the chain up and running. The recommended number of nodes for this implementation is at least 32, but you can use more or less nodes depending on the application’s needs.

## Semi-permissioned

In a semi-permissioned implementation, both Validator types are used:

* A normal Validator, simply called "Validator", similar to the permissionless one and added to the network the same way (by locking tokens); and
* A type of Validator called "Sentinel", similar to the permissioned one and added to the network the same way (with a master address).

This implementation is different in that neither Validators nor Sentinels can create a block on their own. The randomness requires at least one of the transactions from a Sentinel and whoever publishes the block has to follow the Validator list order set during rdPoS processing.

This means the network can have a smaller number of Validators (e.g. 16 instead of 32), requiring less computing power for verification but remaining highly secure. Since Sentinels take part in the process, every extra byte in the concatenated randomness seed will change the resulting hash, thus ensuring that any attempt of tampering is easily detected and dealt with.

