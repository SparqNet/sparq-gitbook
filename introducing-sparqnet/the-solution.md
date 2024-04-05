---
description: How is SparqNet solving the problem?
---

# The Solution

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
