---
description: Cutting out malicious nodes, carefully.
---

# Slashing

What happens when a node answers with a "randomness" hash that does not match its own hash? Or when a node creates an invalid block with invalid transactions? Or when a node can't create a block before reaching the network's time limit?

Misbehaving nodes suffer consequences. Since Validator signatures are required at protocol level, if a Validator tries to break the rules, it's possible to know who it is thanks to the signature, and "slash" it from the network.

At the moment the biggest problem is a group of Validators being "slashed" and halting network activity. This can be solved by adding extra conditions to the network - for example, if the network wants to change the current block creator (in case it's been "slashed"), at least 90% of the Validators in the network have to sign a transaction consenting with the change, always maintaining the majority's consensus.
