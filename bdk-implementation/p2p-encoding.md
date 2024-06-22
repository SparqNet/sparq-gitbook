---
description: How P2P messages are structured in the Blockchain Development Kit (BDK).
---

# P2P Encoding

This subchapter explains how P2P data travels through AppLayer, and how said data is encoded and decoded between nodes. All hexes displayed in this document are interpreted as bytes.

## Handshake

After the session successfully creates a connection, the first thing that happens is a handshake.

| Variable      | Size    | Info            |
| ------------- | ------- | --------------- |
| NodeType      | 1 Byte  | Host node type  |
| P2PServerPort | 2 Bytes | P2P Server Port |

## P2P::Message

Structure that holds any type of P2P message, encoded as follows:

| Variable     | Size    | Info                                        |
| ------------ | ------- | ------------------------------------------- |
| Request Flag | 1 Byte  | Request Type (Answer, request or broadcast) |
| Random ID    | 8 Bytes | Request ID                                  |
| Command ID   | 2 Bytes | Command ID                                  |
| Payload      | X Bytes | Message Command Payload                     |

### Request Flag

Used to tell if the request is a "Request" (0), an "Answer" (1) to a previous request, or a "Broadcast" (2) request (verify and broadcast towards other nodes of the network).

A Request will always wait and have a respective Answer, while Broadcasts are processed directly. See the difference between functions called by `handleAnswer()`/`handleRequest()` and functions called by `handleBroadcast()`. The random ID of the Broadcast request is used to know if the node has previously received that broadcast and if it should rebroadcast or not.

### Random ID

Used to tell different requests apart and make async requests. During a request, the random ID is calculated with 8 random bytes. During an answer, the random ID is equal to the random ID of the respective request. If it's a Broadcast request, the randomID equals `SafeHash(payload)`.

### Command ID

Used to tell which command type is encoded in the payload. See below for more info.

### Payload

Used as the payload of the request/answer (if needed).

### Example

Given the example `0x01adf01827349cad810002123456789abcdef` we can extract the following values:

* Request Flag: `01`
* Random ID: `adf01827349cad81`
* Command ID: `0002`
* Payload: `123456789abcdef`

## Commands

### Ping

Pings another node.

* Command ID: `0000`
* Request Type: Answer or Request
* Request Payload: Empty
* Request Example: `0x00adf01827349cad810000`
* Answer Payload: Empty
* Answer Example: `0x01adf01827349cad810000`

### Info

Requests a basic list of information from another node, while also sending its own.

* Command ID: `0001`
* Request Type: Answer or Request
* Request Payload: node version + epoch + latest nHeight + latest nHash + nConnectedNodes
* Request Example: `0x00adf01827349cad81000100000000000000010005f70436085980000000000000000156aef6ce3d9cefb653ea6a53fc8e810c95268c01428223a8ee267ed2ac9f05d8`
* Answer Payload and Example: same as request

| Variable | Size     | Info                           |
| -------- | -------- | ------------------------------ |
| version  | 8 Bytes  | Node Version                   |
| Epoch    | 8 Bytes  | Timestamp in UNIX microseconds |
| nHeight  | 8 Bytes  | latest block nHeight           |
| Hash     | 32 Bytes | latest block hash              |

### RequestNodes

Request a list of connected nodes to a given node.

* Command ID: `0002`
* Request Type: Answer or Request
* Request Payload: Empty
* Request Example: `0x00adf01827349cad810002`
* Answer Payload: Array of address type + address + port (only connections towards servers)
* Answer Example: node has connections to servers `127.0.0.1:8080` and `127.0.0.1:8081` and gets requested with a `requestNodes`, it will encode an address as following: `Bytes(NodeType) + Bytes(NodeID) + "0x00"/"0x01" depending on address type + Bytes(address) + Bytes(port)`. The encoded string will be `Answer + RandomID + Command ID + Address¹ + Address²`
  * Normal Node at `127.0.0.1:8080`, encoded as `0x00007f0000011f90`
  * Discovery Node at `127.0.0.1:8081`, Node ID = , encoded as `0x01007f0000011f91`
  * Final encoded answer string = `0x00adf01827349cad81000200007f0000011f9001007f0000011f91`

| Variable                  | Size          | Info                            |
| ------------------------- | ------------- | ------------------------------- |
| Node Type (Repeatable)    | 1 Byte        | Node Type (Normal or Discovery) |
| Address Type (Repeatable) | 1 byte        | Address type (v4 or v6)         |
| Address (Repeatable)      | 4 or 16 Bytes | Address                         |
| Port (Repetable)          | 2 Bytes       | Node Server Port                |

### RequestValidatorTxs

Request the TxValidator memory pool of another node.

* Command ID: `0003`
* Request Type: Answer or Request
* Request Payload: Empty
* Request Example: `0x00adf01827349cad810003`
* Answer Payload: nTxCount + \[ 4 Bytes Tx size + N Bytes Tx data ]
* Answer Example: node has two transactions within mempool with the following RLPs: `f86ba4cfffe74621e0caa14dfcb67a6f263391f4a6ad957ace256f3e706b6da07517a63295165701823f44a08cfdf3826149ca0317eaf4f5832c867a4f5050e3e70d635323947d61a4f35618a07dbe86e6a8cef509c7c6174cb8c703ddd8cb695511d72043630d99888ff2ba20`, and `f86ba4cfffe74621e0caa14dfcb67a6f263391f4a6ad957ace256f3e706b6da07517a63295165701823f44a01545c0c89ad5fda9e4c6ef65f264ef575fa2edebef29d577f88d365ff9d28357a00d9ed64e1675315477aca44908148b9b572c7de45d420398193dcfc2d430d158`
  * To encode this, the node has to encode the number of transactions (`0x00000002`), and then append the transactions' sizes and datas in an ordered manner
  * The resulting answer message would be `0x01adf01827349cad810003000000020000006df86ba4cfffe74621e0caa14dfcb67a6f263391f4a6ad957ace256f3e706b6da07517a63295165701823f44a08cfdf3826149ca0317eaf4f5832c867a4f5050e3e70d635323947d61a4f35618a07dbe86e6a8cef509c7c6174cb8c703ddd8cb695511d72043630d99888ff2ba200000006df86ba4cfffe74621e0caa14dfcb67a6f263391f4a6ad957ace256f3e706b6da07517a63295165701823f44a01545c0c89ad5fda9e4c6ef65f264ef575fa2edebef29d577f88d365ff9d28357a00d9ed64e1675315477aca44908148b9b572c7de45d420398193dcfc2d430d158`

| Variable            | Size    | Info                         |
| ------------------- | ------- | ---------------------------- |
| nTx                 | 4 Byte  | Tx Counter                   |
| TxSize (Repeatable) | 4 Bytes | Tx Size                      |
| TxData (Repeatable) | N byte  | Transaction RLP encoded data |

### BroadcastValidatorTx

Broadcast a given TxValidator to all known connected nodes connected. They should validate and rebroadcast.

* Command ID: `0004`
* Request Type: Broadcast
* Request Payload: Tx RLP
* Request Example: `0x02adf01827349cad810004f86ba4cfffe74621e0caa14dfcb67a6f263391f4a6ad957ace256f3e706b6da07517a63295165701823f44a01545c0c89ad5fda9e4c6ef65f264ef575fa2edebef29d577f88d365ff9d28357a00d9ed64e1675315477aca44908148b9b572c7de45d420398193dcfc2d430d158`

### BroadcastTx

Broadcast a given TxBlock to all known connected nodes. They should validate and rebroadcast.

* Command ID: `0005`
* Request Type: Broadcast
* Request Payload: Tx RLP
* Request Example: `0x02adf01827349cad810005f86b02851087ee060082520894f137c97b1345f0a7ec97d070c70cf96a3d71a1c9871a204f293018008025a0d738fcbf48d672da303e56192898a36400da52f26932dfe67b459238ac86b551a00a60deb51469ae5b0dc4a9dd702bad367d1111873734637d428626640bcef15c`

### BroadcastBlock

Broadcast a given Block to all known connected nodes. They should validate and rebroadcast.

* Command ID: `0006`
* Request Type: Broadcast
* Request Payload: Block RLP
* Request Example: `0x02adf01827349cad81000618395ff0c8ee38a250b9e7aeb5733c437fed8d6ca2135fa634367bb288a3830a3c624e33401a1798ce09f049fb6507adc52b085d0a83dacc43adfa519c1228e70122143e16db549af9ccfd3b746ea4a74421847fa0fe7e0e278626a4e7307ac0f600000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000186c872a48300000000057de95400000000000000d918395ff0c8ee38a250b9e7aeb5733c437fed8d6ca2135fa634367bb288a3830a3c624e33401a1798ce09f049fb6507adc52b085d0a83dacc43adfa519c1228e70122143e16db549af9ccfd3b746ea4a74421847fa0fe7e0e278626a4e7307ac0f600000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000186c872a48300000000057de95400000000000000d9`

### RequestTxs

Request the TxBlock memory pool of another node.

* Command ID: `0007`
* Request Type: Answer or Request
* Request Payload: Empty
* Request Example: `0x00adf01827349cad810003`
* Answer Payload: nTxCount + \[ 4 Bytes Tx size + N Bytes Tx data ]
* Answer Example: node has two transactions within mempool with the following RLPs: `f86ba4cfffe74621e0caa14dfcb67a6f263391f4a6ad957ace256f3e706b6da07517a63295165701823f44a08cfdf3826149ca0317eaf4f5832c867a4f5050e3e70d635323947d61a4f35618a07dbe86e6a8cef509c7c6174cb8c703ddd8cb695511d72043630d99888ff2ba20`, and `f86ba4cfffe74621e0caa14dfcb67a6f263391f4a6ad957ace256f3e706b6da07517a63295165701823f44a01545c0c89ad5fda9e4c6ef65f264ef575fa2edebef29d577f88d365ff9d28357a00d9ed64e1675315477aca44908148b9b572c7de45d420398193dcfc2d430d158`
  * To encode this, the node has to encode the number of transactions (`0x00000002`), and then append the transactions' sizes and datas in an ordered manner
  * The resulting answer message would be `0x01adf01827349cad810003000000020000006df86ba4cfffe74621e0caa14dfcb67a6f263391f4a6ad957ace256f3e706b6da07517a63295165701823f44a08cfdf3826149ca0317eaf4f5832c867a4f5050e3e70d635323947d61a4f35618a07dbe86e6a8cef509c7c6174cb8c703ddd8cb695511d72043630d99888ff2ba200000006df86ba4cfffe74621e0caa14dfcb67a6f263391f4a6ad957ace256f3e706b6da07517a63295165701823f44a01545c0c89ad5fda9e4c6ef65f264ef575fa2edebef29d577f88d365ff9d28357a00d9ed64e1675315477aca44908148b9b572c7de45d420398193dcfc2d430d158`

### NotifyInfo

Same as Info, but as a notification instead (doesn't request the other node's info).

* Command ID: `0008`
* Request Type: Notification
* Request Payload: node version + epoch + latest nHeight + latest nHash + nConnectedNodes
* Request Example: `0x03adf01827349cad81000800000000000000010005f70436085980000000000000000156aef6ce3d9cefb653ea6a53fc8e810c95268c01428223a8ee267ed2ac9f05d8`

| Variable | Size     | Info                           |
| -------- | -------- | ------------------------------ |
| version  | 8 Bytes  | Node Version                   |
| Epoch    | 8 Bytes  | Timestamp in UNIX microseconds |
| nHeight  | 8 Bytes  | latest block nHeight           |
| Hash     | 32 Bytes | latest block hash              |

### RequestBlock

Requests data from a given block or range of blocks to another node.

* Command ID: `0009`
* Request Type: Answer or Request
* Request Payload: block height start range + block height end range + bytes limit (minimum 1 block)
* Request Example: TODO
* Answer Payload: serialized block size + content (1..N blocks)
* Answer Example: TODO
