# How to manipulate the database

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
