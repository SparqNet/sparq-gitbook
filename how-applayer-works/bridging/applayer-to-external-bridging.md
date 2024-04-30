---
description: How AppLayer bridges with other blockchains.
---

# AppLayer-to-External Bridging (Ethereum, Solana, etc.)

There are multiple edge cases related to external bridging. For example, it's not possible to natively push data into these chains without paying transaction fees, and those external networks are also limited on both processing power and how much signature verification can be done.

Knowing this, at least for now, the current bridging implementation for them is handled by Sentinels and owned by Sparq Labs and its most trusted partners to ensure operational safety. The contract checks signatures of Validators and Sentinels but only the Sentinels can write into the contracts on these chains.

This method of bridging follows lock/release mechanisms, unless a token is fully integrated with mint/burn bridging.
