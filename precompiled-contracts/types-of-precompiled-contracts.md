---
description: How AppLayer differentiates pre-compiled contracts.
---

### Types of pre-compiled contracts

AppLayer frequently references pre-compiled contracts in three ways: *1st-party*, *3rd-party*, and *AppChains and/or AppLayers*.

**1st-party** contracts are basically the contracts provided by AppLayer itself as ready-to-use templates in the `src/contract/templates` subfolder (e.g. ERC20, ERC721, etc.), officially supported and leveraging all the features of the blockchain. Those contracts are also available in the AppLayer EVM chain.

**3rd-party** contracts act like a "slim L2", in the sense that they are only processed, but have no concepts of consensus, blocks, transactions, etc. like a normal contract would. They essentially act like a "daemon" of sorts, running atop the main chain (just like a Layer 2), reading transactions made on it and reacting accordingly if said transaction happens to call it. The transaction is executed and the contract publishes the results back on the main chain (e.g. a user sent tokens to a 3rd-party exchange contract, once the transaction is confirmed on the main chain the contract will read the data field and execute its own logic, then send the exchanged tokens back on the main chain as its own transaction).

**AppChains and/or AppLayers** act as a "full Layer 2" instead, having their own blocks, transactions, consensus, etc.., but still depending on the main chain to execute their separate logic.
