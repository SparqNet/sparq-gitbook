---
description: How native chains exchange tokens on SparqNet.
---

# OrbiterSDK <--> OrbiterSDK Token Bridging

The same method for arbitrary data bridging is used for token bridging, but there are extra checkups to make sure that a given chain is not minting another chain's tokens.

Due to how the project was designed, when doing a cross-chain transaction we can only ensure that the data _exists_, not that it is _valid in context_. That breach allows a given chain to mint the native token of another chain, because the Gateway Network does not verify if the token is valid inside that network.

We avoid this problem by keeping a "token table", which is just a "spreadsheet" of chains and their external token balances. Of course, this doesn't include the given chain's own native token, since it can freely mint its own token itself and does its own internal validations to avoid invalid minting conditions.

For example, we have chains A, B, and C, each one with tokens of each other. The Gateway Network keeps track of:

* How many B's and C's exist on A
* How many A's and C's exist on B
* How many A's and B's exist on C

When bridging another chain's tokens, the Gateway Network checks if that chain has enough balance to do so. When bridging your own tokens, the Gateway Network only has to increase the balance on the target chain, since the `exit` transaction from your chain has to be included in one of your blocks, which means it has been verified and validated inside your own network, so there's no need to do it again from the outside.

OrbiterSDK <--> OrbiterSDK Bridge follows mint/burn mechanisms.
