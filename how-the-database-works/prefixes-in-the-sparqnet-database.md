# Prefixes in the SparqNet database

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
