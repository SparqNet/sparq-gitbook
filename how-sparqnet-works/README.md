---
description: Exploring the functional elements of SparqNet
---

# How SparqNet works

SparqNet’s functionality is contained in four primary components:

### Validators

A validator is a community member who has set up a node and locked at least 10,000 Sparq tokens through the addValidator transaction, using the validator's address as an argument. They create blocks, the "randomness" seed used to select the next block creator, and are responsible for gathering and signing data on bridging and on blocks.

### Sentinels

Sentinels are similar to Validators, except they cannot create blocks nor act on their own. It is required that both randomly selected validators and sentinels send the same data to the requester, otherwise they will be reported to the network as a malicious node. Sparq Labs Inc. hosts them to ensure that will not happen.

### Subnets&#x20;

Subnets are blockchains built using the SparqNet SDK and can communicate with SparqNet. A subnet primarily enables developers to create a chain dedicated to a particular application, with the chain’s rules tailored to the security, performance, speed, decentralization and other service delivery needs of the application.&#x20;

All the languages supported here compile to a binary. Additionally, whoever sets up a subnet is in charge of validating the transactions on it and attracting others to run validator nodes.&#x20;

### Bridging

An application-specific chain is limited in that it can not support a complete service that involves interaction between more than one application. We address this issue by offering native bridging, where SparqNet serves as an intermediary between two Subnets or dApp chains trying to communicate with each other.

​​It's possible to bridge arbitrary data and tokens, both between SparqNet nodes and between SparqNet and external networks.

#### SparqNet <-> SparqNet Data Bridging

When using SparqNet to bridge two native chains exchanging arbitrary data, the procedure is as follows:

* Subnet A sends a request to SparqNet for Data X from Subnet B. This request is written to the next block and relayed to SparqNet with the block reference.

<figure><img src="../.gitbook/assets/Diagram 1.0.png" alt=""><figcaption></figcaption></figure>

* SparqNet randomly selects a set of Validators and Sentinels from the network using RandomGen (which is the same in all nodes in the network). The selected Validators and Sentinels check Subnet A’s request and relay it to Subnet B.&#x20;

<figure><img src="../.gitbook/assets/Diagram 2.png" alt=""><figcaption></figcaption></figure>

* Subnet B receives the request and gathers the data inside a merkled item within its blockchain, then sends it to SparqNet with its reference (its location on Subnet B).

<figure><img src="../.gitbook/assets/Diagram 3.png" alt=""><figcaption></figcaption></figure>

* The Validators and Sentinels then check the Data X against other nodes of Subnet B to confirm its presence in the mentioned block. &#x20;

<figure><img src="../.gitbook/assets/Diagram 4.png" alt=""><figcaption></figcaption></figure>

* Once confirmed, these Validators and Sentinels sign the data, publish it within SparqNet and send it back to Subnet A along with its signature.

<figure><img src="../.gitbook/assets/Diagram 5.png" alt=""><figcaption></figcaption></figure>

* Subnet A verifies the signature and checks whether the randomly selected nodes used the network’s RandomGen seed. If everything matches, the exchange is complete, but if not, someone is being malicious and can be reported to the network.

#### SparqNet <-> SparqNet Token Bridging

The same method for arbitrary data bridging is used for token bridging, but there are extra checks to guarantee that a given Subnet is not minting another Subnet's tokens. Due to the system's design, when doing a cross-chain transaction, we can only ensure that the data exists, not that it is valid in context.

Therefore, this loophole allows a given Subnet to mint the native token of another Subnet, because SparqNet does not verify if the token is valid inside that network. We avoid this problem by keeping a token table of the Subnets. This is a simple tabulation of the subnets and their token balances.

However, this catalog does not include the given Subnet's native token since the Subnet can freely mint its native token in accordance with its internal checks, so we do not need to keep it on the table. Instead, the table focuses on how many of another Subnet’s tokens are on a specific Subnet.

For example, we have Subnet A, B, and C, each one with tokens of each other, where SparqNet keeps track of:

* How many B's and C's exist on A
* How many A's and C's exist on B
* How many A's and B's exist on C

When bridging another Subnet's tokens, SparqNet checks if that Subnet has enough balance to do so. When bridging your own tokens, SparqNet only has to increase the balance on the target Subnet, since the exit transaction from your Subnet has to be included in one of your blocks, which means it has been verified and validated inside your own network, thus there's no need to verify and validate it again from the outside.

SparqNet <-> SparqNet Bridge follows mint/burn mechanisms.

**How is safety ensured?**

The nodes that read from a given Subnet are determined using RandomGen, the trustless decentralized randomness generator developed by Sparq Labs Inc.. We ensure to keep a fair selection of nodes, however, there is a possibility of a 51% attack.

For example, in a network with 100 nodes, if a malicious user controls 50 of them, and all of them get selected for driving a cross-chain request and a block, they could collude and forward any message they want.

We avoid this by introducing Sentinels to the network. Sentinels are Sparq Labs powered Validators that ensure this collusion does not happen. Sentinels can not create new blocks, but rather work together with Validators to fortify the network’s security.

#### SparqNet <-> External Bridging (Ethereum, Solana, etc.)

Due to the limited processing power of these networks, the current bridging implementation for them is semi-centralized and owned by Sparq Labs Inc. and its approved partners. The contract checks signatures of validators and sentinels but only the sentinels can write into the contracts on these chains.

SparqNet <-> External bridge follows lock/release mechanisms unless a token is fully integrated with SparqNet mint & burn bridging.\
