---
description: The hydra we're trying to slay
---

# The Problem With EVMs

Chains with virtual machines share the same problem: contracts built on top of them often have limited speed and flexibility due to the nature of the virtual machines themselves, as they're built to be "generic computers with limited throughput".

This is common on Ethereum and other EVM-based chains - having to share a "generic computer" with the whole world is *inefficient by design*. Forcing a chain to be both generic and decentralized at the same time puts heavy limits on which types of applications you can decentralize and how much you can decentralize them.

## Limitations of the EVM

For example, in the case of the Ethereum Virtual Machine, you can not do any of the following:

* Loop a function more than 50 times due to block gas limit constraints;
* Have a stack size larger than 16 variables due to constraints on the EVM itself;
* Parallelize multiple contract calls (as in, every time a new block has multiple transactions that interact with multiple different contracts, you have to load the contract, parse and save changes to the database for *each* single one of these contracts, *in order*).

As quoted by [Itamar](https://github.com/itamarcps): *"The biggest problem is that everyone is sharing the same computer, and that computer is a Commodore 64"*.

## The solution

If the problem is inherently tied to the EVM, then... *why don't we just get rid of it?* This seems like a reasonable solution, but it can't be the only one either.

The current blockchain landscape is primarily dominated by the Solidity EVM, and for many Web3 companies it is unrealistic to completely remove it from existing applications. However, to properly scale a Web3 based application, we must venture outside the virtual machine.

Our solution is a hybrid of old and new: a modular blockchain that not only allows for *natively-coded contracts*, but also a natively-coded, performance-centric EVM with *stateful pre-compiles*. In other words, a blockchain that doesn't *need* a virtual machine to run smart contracts, but still has support for existing contracts to be deployed as-is.

Having an EVM running in parallel of stateful pre-compiles unlocks performance optimizations to existing Solidity contracts and accelerates them with pre-compiled functions within the state, programmed in common performance-driven development languages, such as C++, C#, Rust, and more.

## The caveats (and how we're solving them)

There’s a tremendous advantage to using native stateful pre-compiles, however, this approach also comes with its own set of problems:

* It can be tricky to push new smart contract code into the network as a pre-compile. If you want to add or remove logic from your contract in the network, you have to force a mandatory update to all node operators - this could take *days*, which is a pretty big deal depending on how mission-critical the update is
* An application-specific pre-compiled contract might be too expensive to be natively executed on 1st party validation
* If the pre-compile is not hard-coded into the network and validated from a 3rd party, it must be ensured that this communication remains secure

What if there was a blockchain system that incorporated stateful pre-compiles, allowing third parties to deploy and natively support these contracts within a single network that shares its state?

This is exactly what AppLayer does. AppLayer is a modular blockchain of multiple layers, including an EVM, stateful pre-compiles, and chain abstraction.

