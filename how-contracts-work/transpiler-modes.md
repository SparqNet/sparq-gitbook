---
description: In which modes you can use the Transpiler
---

# Transpiler Modes

The Solidity to C++ Transpiler can be used with any type of Solidity source code as long as it's compatible, but there are different ways a developer can use the transpiler. These are the basic mode and advanced mode.

In the basic mode, the developer only declares the functions and local variables to be used, so the transpiler can create argument encoding and database functions necessary for the structure of the contract. This is the recommended way since you will be coding the contract logic in C++.

In the advanced mode, the developer can input a full Solidity contract and it will convert all the logic in the implementation into C++ source code. Of course, as the application grows and starts getting more complex, the Solidity source code and transpiled code are not enough for the performance requirements, and this is where the freedom of C++ shines.
