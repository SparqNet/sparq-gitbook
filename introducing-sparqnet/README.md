---
description: A quick introduction to what SparqNet does
---

# Introducing SparqNet

### The Problem

Smart contracts often lack speed and flexibility when running on blockchains with Virtual Machines (VMs). This is common on Ethereum and other EVM-based chains where everyone shares one generic computer with limited throughput.

This design is inefficient because it forces the chain to be both generic and decentralized. This limits the types of applications you can decentralize and how much you can decentralize them.

For example, in the case of the Ethereum Virtual Machine, you can not do the following:

* Loop a function more than 50 times due to the block gas limit.
* Have a stack size larger than 16 variables due to constraints on the EVM itself.
* Parallelize multiple contract calls. Basically, every time a new block has multiple transactions that interact with multiple different contracts, you have to load the contract, parse and save changes to the database for each single one of these contracts, in order.

### The Solution

Using a natively-coded blockchain enables you to apply performance optimizations to the code for a specific application's implementation. You can even do this in common performance-driven development languages, such as C++, C#, Rust, Go, and others. \


The SparqNet project provides developers with tools and documentation to easily create application-specific Web 2.0 chains with unprecedented freedom.

### The Caveats (and Solution)

While there’s tremendous upside to implementing the solution with a native and application-specific blockchain, this approach also comes with some problems. For example:

* It can be tricky pushing new smart contract code into the network. If you want to include or exclude logic from your contract in your network, you have to initiate a mandatory update through all your node operators, which could take days.
* An application-specific chain supports only one type of application. This would make use cases like DeFi impossible since most DeFi projects typically interact with each other’s contracts to provide a complete service.

For example, suppose you have deployed a AAA game with NFTs in your network but forgot to add an NFT marketplace. Now your users have to wait for days as you implement a marketplace and get node operators in the network to update their nodes to comply with your new implementation.

Accordingly, what if you could natively bridge these assets between networks? This is what SparqNet does. Subnets use the SparqNet as a middleman to communicate with each other, so neither network has to synchronize and verify each other's data completely.

### What is SparqNet?

The SparqNet project is made up of two parts:

* An SDK for developers to easily build their application-specific native chains.
* A network that allows the bridging of data and assets between these app-chains and external chains.

### Potential applications

SparqNet provides all the essential tools to build a wide range of products for decentralized finance, data storage, gaming, and much more.

**DeFi**

Build financial products with security and ease. Leveraging SparqNet's performance network enables a whole new generation of DeFi products such as facilitating millions of trades per second.

**Data Storage**

SparqNet makes data storage easier, more affordable, and secure. Builders can store backups of whole ledgers or fully decentralize any form of a database natively in the Sparq network.

**GameFi**

Gaming projects now have performance infrastructure to build game engines capable of leveraging a pure Blockchain solution with a whole new range of in-game features.

**Potential use cases include:**

1\. Multiplayer games/servers

2\. Decentralized exchange

3\. Decentralized and hyper available caching for dApps’ databases

4\. Decentralized e-mail

5\. HR portal

6\. VPN

7\. Cloud services

8\. Video rendering

9\. E-commerce

10\. Arbitrage bot

11\. Blockchain-enabled utilities such as water and power

12\. Supply chain and logistics
