---
description: How SparqNet implements rdPoS
---

# rdPoS implementation

With the current code, the BlockManager class maintains and applies all the rdPoS logic. The "randomness" engine is in utils/random.h and also includes vector sorting. At the moment, there is a prototype of a centralized implementation.
