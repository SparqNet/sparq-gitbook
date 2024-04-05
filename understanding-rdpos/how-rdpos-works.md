---
description: How SparqNet uses rdPoS
---

# How rdPoS works

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
