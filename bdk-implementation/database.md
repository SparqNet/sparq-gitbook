---
description: How Blockchain Development Kit's (BDK) internal database is structured and how data is stored in it.
---

# Database

BDK validators use an in-disk database for storing data about themselves and other nodes in the network, such as block/transaction data, contract data, node metadata, etc. Depending on the component (e.g. Storage), they might have their own database folder reserved just for them.

The database itself is an abstraction of a [Speedb](https://github.com/speedb-io/speedb) database - a simple key/value database, but handled in a different way: keys use *prefixes*, which makes it possible to batch read and write, so we can get around the "simple key/value" limitation and divide data into sectors.

The database requires a filesystem path to open it (if it already exists) or create it on the spot (if it doesn't exist) during construction. It closes itself automatically on destruction.

Content in the database is stored as raw bytes. This is due to space optimizations, as one raw byte equals two UTF-8 characters (e.g. an address like `0x1234567890123456789012345678901234567890`, ignoring the "0x" prefix, occupies 20 raw bytes - "12 34 56 ..." - , but 40 bytes if converted to a string, since each byte becomes two separate characters - "1 2 3 4 5 6 ...").

For the main CRUD operations, refer to the `has()`, `get()`, `put()` and `del()` functions. Due to how the database works internally, updating an entry is the same as inserting a different value in a key that already exists, effectively replacing the value that existed before (e.g. `put(oldKey, newValue)`). There's also `getBatch()` and `putBatch()` for batched operations, as well as `getKeys()` for fetching only the keys, and `keyFromStr()` for encapsulating a key into a Bytes object.

## Structs and Prefixes

We have three helper structs to ease database manipulation:

* `DBServer` - struct that contains the host and version of the database that will be connected to
* `DBEntry` - struct that contains an entry to be inserted or read by the database, and has only two members: key and value, both strings
* `DBBatch` - struct that contains multiple `DBEntry`s to be inserted and/or deleted all at once

We also have a `DBPrefix` namespace to reference the database's prefixes in a simpler way:

| Descriptor      | Prefix |
| --------------- | ------ |
| blocks          | 0x0001 |
| blockHeightMaps | 0x0002 |
| nativeAccounts  | 0x0003 |
| txToBlocks      | 0x0004 |
| rdPoS           | 0x0005 |
| contracts       | 0x0006 |
| contractManager | 0x0007 |
| events          | 0x0008 |
| vmStorage       | 0x0009 |
| txToAddr        | 0x000A |

Those prefixes are concatenated to the start of the _key_, so an entry that would have, for example, a key named "abc" and a value of "123", if inserted to the "0003" prefix, would be like this inside the database (in raw bytes format, strings here are just for the sake of the explanation): `{"0003abc": "123"}`

## Prefixes Overview

### blocks

Used to store serialized blocks based on their hashes.

| Key                | Value            |
| ------------------ | ---------------- |
| Prefix + BlockHash | Serialized Block |

### blockHeightMaps

Used to store block hashes based on their heights.

| Key                                   | Value     |
| ------------------------------------- | --------- |
| Prefix + Padded uint64\_t BlockHeight | BlockHash |

Padded means that the `uint64_t` is padded with 0's to the left to ensure a fixed length of 8 bytes.

### nativeAccounts

Used to store serialized native accounts ("balance + nonce") based on their addresses.

| Key              | Value                    |
| ---------------- | ------------------------ |
| Prefix + Address | Serialized NativeAccount |

Serialization for a native account goes like this: `requiredBytes(balance) + bytes(balance) + requiredBytes(nonce) + bytes(nonce)`.

For example, an account with balance 1000000 and nonce 2 would be serialized as `03 + 0f4240 + 01 + 02`.

An account with balance 0 and nonce 0 would be serialized as `0000`.

### txToBlocks

Used to store block hashes, the tx indexes within that block and the block heights, based on their transaction hashes.

| Key                      | Value                                                                  |
| ------------------------ | ---------------------------------------------------------------------- |
| Prefix + TransactionHash | BlockHash + Padded uint32\_t BlockIndex + Padded uint64\_t BlockHeight |

Padded means that the `uint32_t` and `uint64_t` are padded with 0's to the left to ensure a fixed length of 4 and 8 bytes respectively.

### rdPoS

Used to store Validator addresses based on their index within the rdPoS list.

| Key                                      | Value   |
| ---------------------------------------- | ------- |
| Prefix + Padded uint64\_t ValidatorIndex | Address |

### contracts

Used in a multitude of ways, where an "additional" prefix is used per contract (the contract address).

| Key                                           | Value                        |
| --------------------------------------------- | ---------------------------- |
| Prefix + Contract Address + "contractName"    | The Contract Class Name      |
| Prefix + Contract Address + "contractAddress  | The Contract Address         |
| Prefix + Contract Address + "contractCreator" | The Contract Creator Address |
| Prefix + Contract Address + "contractChainId" | The Contract ChainId         |

Contracts are free to use the contract prefix as they wish, but the following is the default structure for the contract prefix:

| Key                                                      | Value                                |
| -------------------------------------------------------- | ------------------------------------ |
| Prefix + Contract Address + "VARIABLE\_NAME\_AS\_STRING" | The Contract Variable to store in DB |

### contractManager

Used to store a contract class name based on their address.

| Key                       | Value               |
| ------------------------- | ------------------- |
| Prefix + Contract Address | Contract Class Name |

### events

Used to store events emitted from contracts.

| Key                                                                                                              | Value                                  |
| ---------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| Prefix + Padded uint64\_t Block Index + Padded uint64\_t Tx Index + Padded uint64\_t Log Index + ContractAddress | Event data serialized as a JSON string |

### vmStorage

Used to store EVM-related stuff like storage keys and values (essentially the EVM's "permanent memory").

| Key                                         | Value                    |
| ------------------------------------------- | ------------------------ |
| Address (20 bytes) + Storage Key (32 bytes) | Storage Value (32 bytes) |

### txToAddr

Used to store EVM transactions that created contracts, storing the transaction hash and the address where the contract was deployed.

| Key                      | Value              |
| ------------------------ | ------------------ |
| Prefix + TransactionHash | ContractAddress    |

