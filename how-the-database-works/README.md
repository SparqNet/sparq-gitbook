---
description: How the database is structured on SparqNet
---

# How the database works

Since the Subnet, up until the moment of writing this document, is running inside a sandbox and interfacing with AvalancheGo's VM, it's not possible to use our own database. We have to use the database provided by AvalancheGo via gRPC.

The database itself is a simple key/value database similar to Google's [LevelDB](https://github.com/google/leveldb) (in fact we're using it internally), but modified so that it's possible to batch read and write using a logic structure based on prefixes.

### Prefixes

The database has the following prefixes:

```
0001 -- Key: Block Hash | Value: Block
```

```
0002 -- Key: Block nHeight | Value: Block Hash
```

```
0003 -- Key: Tx Hash | Value: Transactions
```

```
0004 -- Key: Address | Value: Native Balance + nNonce
```

```
0005 -- ERC20 Tokens/State
```

```
0006 -- ERC721 Tokens/State
```

```
0007 -- Key: Tx Hash | Value: Block Hash
```

Those prefixes are concatenated to the start of the _key_, so an entry that would have, for example, a key named `"abc"` and a value of `"123"`, if inserted to the `"0003"` prefix, would be like this inside the database: `{"0003abc": "123"}`

### How to manipulate the database

In the database implementation, there's a DBPrefix to reference each prefix in a simpler way:

```
0001 = DBPrefix::blocks
```

```
0002 = DBPrefix::blockHeightMaps
```

```
0003 = DBPrefix::transactions
```

```
0004 = DBPrefix::nativeAccounts
```

```
0005 = DBPrefix::erc20Tokens
```

```
0006 = DBPrefix::erc721Tokens
```

```
0007 = DBPrefix::TxToBlocks
```

```
0008 = DBPrefix::validators
```

There are also three structs, namely:

* `DBServer` - struct that contains the host and version of the database that will be connected to.
* `DBEntry` - struct that contains an entry to be inserted or read by the database and has only two members: key and value, which are both strings.
* `WriteBatchRequest` - struct that contains multiple `DBEntrys` to be inserted and/or deleted all at once.

### The DBService class and its members

The class that abstracts the database itself and its operations is called DBService. The constructor requires a file system path, it opens the database (if it exists) or creates it on the spot (if it doesn't exist) at that moment.

To close the database, call the `DBService::close` function. Aside from using the structures above, it also uses an internal pointer to a `leveldb::DB` object and a group of member functions that abstract the main CRUD operations for LevelDB:

* `DBService::has` - checks if a key exists in a given database prefix.
* `DBService::get` - gets a value from a key in a given database prefix, if it exists.
* `DBService::put` - inserts a key and value in a given database prefix. Due to the way LevelDB works, updating an entry is the same as inserting a different value in a key that already exists.
* `DBService::del` - deletes a key in a given database prefix.
* `DBService::writeBatch` - same as put + del but batched. The function requires a `WriteBatchRequest` struct, therefore, all operations in it are done in one go.
* `DBService::readBatch` - same as get but returns all entries in a given database prefix. This function has two overloads - the first one returns all entries and requires only the prefix, and the second one returns only values from specific keys and requires both the prefix and a key list.
* `DBService::removeKeyPrefix` - helper function that removes the prefix from a given key (e.g. key `"0003abc"` with value `"123"`, after going through this function it would return `"abc"`).
