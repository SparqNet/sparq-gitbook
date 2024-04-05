---
description: How blocks are structured on SparqNet
---

# Block structure

Conceptually, a block has the following structure:

* Validator signature
* Header, containing:
*
  * The previous block hash (a hash of the whole header, signed by the Validator)
  * The "Randomness"
  * A Validator Merkle Tree (to verify the integrity of Validator transactions)
  * A Block Merkle Tree (to verify the integrity of block transactions)
  * A UNIX timestamp of the block in nanoseconds
  * The Block height (nHeight)
* Content:
*
  * Number of Validator transactions
  * Number of block transactions
  * List of Validator transactions
  * List of block transactions

In practice, a block is simply a serialized bytes string, transmitted through the network and stored in the blockchain, therefore it's up to the code logic to parse said details. The above structure can be interpreted in a simpler form as follows:

```cpp
  65 bytes - Validator signature
Header:
  32 bytes - Previous block hash
  32 bytes - "Randomness"
  32 bytes - Validator Merkle Tree
  32 bytes - Block Merkle Tree
  8 bytes - Timestamp
  8 bytes - Block height
Content:
  8 bytes - Number of Validator transactions
  8 bytes - Number of block transactions
  8 bytes - Offset of Validator transactions array
  8 bytes - Offset of block transactions array
  [
    4 bytes - Validator transaction size
    N bytes - Validator transaction
    ...
  ]
  [
    4 bytes - Block transaction size
    N bytes - Block transaction
    ...
  ]
```

Blocks transmitted on the network must have the Validator signature, which is part of the block but is outside its structure. This is done intentionally so the block header can be hashed and signed without interference from the signature itself.
