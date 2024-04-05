---
description: How blocks work on the SparqNet protocol
---

# How blocks work

Blocks on SparqNet are abstracted by the Block class in core/block. The class only has the structure and data of a block. It doesn't perform any kind of operation, validation or verification on the data.

### Block structure

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

### Raw block header analysis

That said, a "raw" block would look like this:

`5c37d504e9415c3b75afaa3ad24484382274bba31f10dcd268e554785d5b807500000181810eb6507a8b54dfbfe9f21d00000001000000aff8ad81be850c92a69c0082e18c94d586e7f844cea2f87f50152665bcbc2c279d8d7080b844a9059cbb00000000000000000000000026548521f99d0709f615aa0f766a7df60f99250b00000000000000000000000000000000000000000000002086ac351052600000830150f7a07e16328b7f3823abeb13d0cab11cdffaf967c9b2eaf3757c42606d6f2ecd8ce6a040684c94b289cdda2282...`

Where:

* Validator signature = `5c37d504e9415c3b75afaa3ad24484382274bba31f10dcd268e554785d5b807500000181810eb6507a8b54dfbfe9f21d00000001000000aff8ad81be850c92a69c`
* Previous block hash = `0082e18c94d586e7f844cea2f87f50152665bcbc2c279d8d7080b844a9059cbb`
* Randomness = `00000000000000000000000026548521f99d0709f615aa0f766a7df60f99250b`
* Validator Merkle Tree = `00000000000000000000000000000000000000000000002086ac351052600000`
* Block Merkle Tree = `830150f7a07e16328b7f3823abeb13d0cab11cdffaf967c9b2eaf3757c42606d`
* Timestamp = `6f2ecd8ce6a04068`
* Block height = `4c94b289cdda2282`
