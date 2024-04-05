---
description: How transactions are parsed on SparqNet
---

# Transaction parsing

There are two ways a transaction can be parsed from a bytes string:

* **Directly from Recursive Length Prefix (RLP) – a data encoding method used by Ethereum. Please note that**:
* RLP requires deriving the "from" account and a validity check using Secp256k1
* The transaction isn’t included in a block, which means it's a new transaction coming from the network
* This method is equivalent to Ethereum's "`rawTransaction`"
* **Directly from the database. Please note that:**
* The transaction is considered trustworthy since it already went through the process above
* The transaction is included in a block, therefore it's part of the blockchain
