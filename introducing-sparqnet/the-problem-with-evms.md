---
description: The hydra we're trying to slay
---

# The Problem With EVMs

Chains with virtual machines share the one same problem: contracts built on top of them often have limited speed and flexibility due to the nature of the virtual machines themselves, as they're built to be "generic computers with limited throughput".

This is common on Ethereum and other EVM-based chains - having to share said "generic computer" with the whole world is _inefficient by design_. Forcing a chain to be both generic and decentralized at the same time puts heavy limits on which types of applications you can decentralize and how much you can decentralize them.

#### Limitations of the EVM

For example, in the case of the Ethereum Virtual Machine, you can not do any of the following:

* Loop a function more than 50 times due to block gas limit constraints;
* Have a stack size larger than 16 variables due to constraints on the EVM itself;
* Parallelize multiple contract calls (as in, every time a new block has multiple transactions that interact with multiple different contracts, you have to load the contract, parse and save changes to the database for _each_ single one of these contracts, _in order_).

As quoted by [Itamar](https://github.com/itamarcps): _"The biggest problem is that everyone is sharing the same computer, and that computer is a Commodore 64"_.

#### The solution

If the problem is inherently tied to the EVM, then... _why don't we just get rid of it?_

Here is our solution: _native blockchains_.

By having a natively-coded blockchain (as in, a blockchain that doesn't need a virtual machine to run smart contracts), it becomes possible to apply performance optimizations to the code for a specific application's implementation, while also providing the flexibility of developing in common performance-driven development languages, such as C++, C#, Rust, Go, and others.

#### The caveats (and how we're solving them)

While there’s a tremendous advantage to implementing a native and application-specific blockchain, this approach also comes with its own set of problems:

* It can be tricky to push new smart contract code into the network. If you want to add or remove logic from your contract in your network, you have to force a mandatory update to all your node operators - this could take _days_, which is a pretty big deal depending on how mission-critical the update is to your ecossystem
* An application-specific chain supports _only one_ type of application. This would make use cases like DeFi impossible in theory, since most DeFi projects typically depend on interaction with each other’s contracts to provide a complete service - e.g. you have deployed a AAA game with NFTs in your blockchain, but forgot to add an NFT marketplace, now your users have to wait for days until A) you implement a marketplace, and B) every node operator in your network actually updates their nodes to comply with your implementation

But what if you could _natively bridge_ these assets between networks? This is what SparqNet does. Sparq-enabled blockchains use the SparqNet network as a middleman to communicate with each other, so neither chain has to sync and verify each other's data completely.
