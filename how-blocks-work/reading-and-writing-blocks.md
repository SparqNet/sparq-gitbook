---
description: How to read and write blocks on SparqNet
---

# Reading and writing blocks

The SparqNet blockchain is abstracted by the `ChainHead` class in `core/chainHead`. The class maintains a collection of blocks approved and validated by the network, other nodes, or itself. Those blocks store transactions, contracts, accounts, and can't be altered once they're in the blockchain, they can only be searched/read from.

Writing new blocks to the blockchain is made by the members of the `ChainTip` class. The `ChainHead` class only receives those blocks and their transactions and stores them permanently in a database.

Searching/reading those blocks is done in several places in the system, so we can conclude that `ChainHead` and database together are the endpoint of the ecosystem's operations history.

### Initialization and class members

The constructor of the `ChainHead` class receives an instance of the database (`DBService dbServer`) during the Node initialization in `Subnet::initialize`. On Subnet initialization, the `Chainhead` loads a history of up to 1000 of the most recent blocks, stored in a previous initialization, from an internal database.

If there are no blocks (e.g. the blockchain has just been deployed and initialized for the first time), a "genesis" block is automatically created and loaded in memory. When reaching 1000 blocks in memory, or after a certain time, older blocks are periodically saved to the database. This makes the blockchain lightweight memory-wise, thus extremely responsive.

This is done by an automatic backup routine (`periodicSaveToDB`), initialized along with the class every 15 seconds. Every method except `push_back`, `pop_back` and `dumpToDB` does not change the blockchain, they only search or read transactions and/or blocks.

* `push_back` - adds a new block accepted by the network to the end of the blockchain.
* `pop_back` - removes the last block accepted by the network from the end of the blockchain.
* `exists` - checks if a block exists in the blockchain. It has two overflows; check by block Hash or by block height (`nHeight`).
* `getBlock` - returns a block from the blockchain. It has two overflows; check by block Hash or by block height (`nHeight`)
* `hasTransaction` - checks if a transaction exists in a block in the blockchain based on its Hash.
* `getTransaction` - returns data from a transaction in a block in the blockchain based on its Hash. It uses the `hasTransaction` method internally.
* `getBlockFromTx` - returns the block that contains a given transaction, based on its Hash.
* `latest` - returns the most recent block approved by the network.
* `periodicSaveToDB` - calls the `dumpToDB` function every 15 seconds.
* `dumpToDB` - stores all blocks in memory in the database. This method is also called on Node shutdown (`Subnet::stop`).
