---
description: How bridging works on SparqNet.
---

# Bridging

There are inherent flexibility problems with **native** and **application-specific** chains when compared to traditional EVM chains. An application-specific chain is limited in that it can not support a complete service that involves interaction between more than one application. Those problems could heavily damage the reputation of a project built on them.

Our solution to this issue is to allow Sparq-enabled blockchains to natively communicate with each other by using the Gateway Network as a middleman, where SparqNet serves as an intermediary between two Subnets or dApp chains trying to communicate with each other. We call that **bridging**.

It's possible to bridge _arbitrary data_ and _tokens_, both _between OrbiterSDK nodes_ and _between OrbiterSDK and external networks_.

**How is safety ensured?**

The Validator and Sentinel nodes that read from a given chain are determined using RandomGen, the trustless decentralized randomness generator developed by Sparq Labs Inc.. We ensure to keep a fair selection of nodes, however, there is a possibility of a 51% attack.

For example, in a network with 100 nodes, if a malicious user controls 50 of them, and all of them get selected for driving a cross-chain request and a block, they could collude and forward any message they want.

We avoid this by introducing Sentinels to the network. Sentinels are Sparq Labs powered Validators that ensure this collusion does not happen. Sentinels can not create new blocks, but rather work together with Validators to fortify the network’s security.
