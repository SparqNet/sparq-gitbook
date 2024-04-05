---
description: How SparqNet implements Merkle and Patricia Trees
---

# Merkle and Patricia Trees

The SparqNet project contains a custom implementation of Merkle and Patricia trees, adapted from the following sites:

* [https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-tree-b8badd6d9591](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-tree-b8badd6d9591)
* [https://lab.miguelmota.com/merkletreejs/example/](https://lab.miguelmota.com/merkletreejs/example/)

Those structures are frequently called "Merkle"/"Patricia", "Merkle Trie"/"Patricia Trie", "Merkle Tree"/"Patricia Tree", etc.

### Concepts

It's recommended to read the articles to better understand the concepts, but in short, they're both data structures in binary tree format (e.g. "heap sort"), where data is stored on the "leaves" and the "branches" are paths to reach said data.

The difference between both structures is in the behavior of the "branches" and the "root":

* A Merkle tree is mainly used for verification - it hashes the previous layers in pairs to make new layers, bottom-up, until it reaches a single result which would be the "root" of the tree. This makes the root a unique fingerprint for the tree as a whole, so you only need to check the root hash to verify that the tree and its leaves are intact.
* A Patricia Tree is mainly used for storage - it breaks the leaf's data into several pieces and uses them to build a branch path that leads to the leaf. For example, a string can be cut into individual characters, and every character becomes a branch node, where the whole branch becomes a "key" that leads to a unique value.

The Ethereum project uses a customized combination of both trees, called MPT ("Merkle Patricia Tree"), for several purposes, such as storing the database state, transaction receipts, and more.&#x20;

SparqNet's objective with the implementation of both trees is to emulate this combination, uniting the benefits of both structures.

### Implementations and class members

SparqNet implements those structures separately in two classes, namely: Merkle and Patricia. Their nodes are abstracted by the classes Hash (for Merkle, declared in Utils) and PNode (for Patricia, declared internally by the class itself for exclusive use - the class still uses Hash for inserts, searches and deletions).

The Merkle class:

* Has tree members that form the tree - the root (root), the complete tree with all layers bottom-up/from leaves to root (layers), and the leaves (leafs).
* The constructor requires a leaf vector, a std::map of block transactions, or a std::map of Validator transactions.
* Each member has its getter - `root()`, `layers()`, and `leafs()`.
* Allows you to check the integrity of one of the leaves using the `getProof()` method, which returns the verification path of a given leaf, bottom-up.

The Patricia class:

* Has only one member that forms the tree - a PNode containing the root of the tree.
* Each PNode has a single character that serves as an ID and a vector of child PNodes, therefore, we only need the root to go through the whole tree and reach the leaves (the constructor hardcodes the root tree to the / character ID).
* The tree can be manipulated with the functions add(), get() and remove()
*
  * `add()` inserts a value in the tree based on a certain Hash
  * `get()` searches a value inside a certain Hash
  * `remove()` deletes a value inside a certain Hash (the branch nodes themselves are NOT deleted, only their data)
