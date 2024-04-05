---
description: What's in the pipeline for the Solidity Transpiler
---

# Upcoming Features

The developer is free to do whatever they want with their contract. For example, they can load it directly into the state without contractManager and without having every variable already loaded in the state. All they have to do is pay attention to the lines.

However, some Solidity features are NOT available on SparqNet (through the transpiler), such as:

* Interfaces¹
* Inline Assembly
* Solidity Version < 0.8
* Libraries²
* Some of Solidity global variables (such as basefee, gasleft, and others)

Interfaces are not supported, but you can include a Contract directly instead of using interfaces. And fortunately, Libraries will be implemented in the future. The development of the Solidity to C++ project will follow simple steps.

We will eventually provide more Solidity features once we adapt them in a static contract manner, so in the meantime, you can examine the Solidity compiler and use its Abstract Syntax Tree.
