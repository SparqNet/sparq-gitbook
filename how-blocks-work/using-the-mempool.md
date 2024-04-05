---
description: How the mempool on SparqNet works
---

# Using the mempool

The mempool on SparqNet is abstracted by the Mempool class in core/chainTip. The class controls the blocks received through the network and contains information such as the preferred block (chosen by the network) and the block status (is it being processed, was it approved or rejected).

When a block reaches consensus (goes from being "processed" to being "approved" or "rejected"), it is removed from the mempool and properly discarded (if rejected) or moved to the blockchain (if approved).

A block that was already received can't be accepted anymore. This applies to every status a block can be in, and to prevent that from happening, the status of all received blocks is stored in a list (`cachedBlockStatus`).

### Initialization and class members

The class does not have a defined constructor and it’s initialized in a pointer in `Subnet::initialize`. The class has the following internal variables:

* `Hash preferedBlockHash` - the preferred block
* `unordered_map<Hash, Block*, SafeHash> internalChainTip` - the mempool itself.
* `unordered_map<Hash, BlockStatus, SafeHash> cachedBlockStatus` - the list of all statuses from received blocks.

Subsequently, during the process of accepting a block, the stored data is copied to a new block and added to the blockchain.

The member functions of the class are:

* `setBlockStatus` - changes the status of a block in `cachedBlockStatus`.
*
  * NOTE: This method is not called anywhere in the code.
* `getBlockStatus` - return the state of a block in `cachedBlockStatus`, if it exists in the mempool.
* `processBlock` - add the block received from the network to the `internalChainTip` and `cachedBlockStatus` lists, to be processed in the next network operations.
* `isProcessing` - verifies if a block is being processed, returning the block state if found, or false if the block isn’t found or isn’t being processed (`BlockStatus::Processing`)
* `accept` - accepts a given block according to the network's request.
*
  * If the block was verified before and is not being processed, it is sent to `State::processNewBlock` and added to the blockchain.
* `reject` - rejects a given block according to the network's request.
* `exists` - verifies if the block exists in `internalChainTip`.
* `getBlock` - returns a block added to the mempool before, or an uncaught exception if it doesn't exist.
* `getPreference` - returns the Hash of the best candidate block selected by the network, or an uncaught exception if it doesn't exist.
* `setPreference` - changes the best block selected by the network.
