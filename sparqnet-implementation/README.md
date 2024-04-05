---
description: How the functional elements of SparqNet interact
---

# SparqNet implementation

### Components

The SparqNet blockchain structure involves interactions between the following components:

* Transaction - implemented as `Tx` in `utils/transaction`
* Block - implemented as `Block` in `core/block`
* Validator - implemented as `Validator` in `core/blockmanager`
* BlockManager - implemented as `BlockManager` in `core/blockmanager`
* Mempool - implemented as `Mempool` in `core/chainTip`
* Chain - implemented as `Chain` in `core/chainHead`

### Validator

The Validator class is an abstraction of a validator node. A Validator node validates batches of transactions on the network and records them as blocks.

### BlockManager

The BlockManager class manages block creation, congestion and validation. The process is handled by an internal list of Validators, of which one is selected to create the block and the others validate it using signatures.

\
