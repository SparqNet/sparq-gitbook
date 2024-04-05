---
description: How native chains exchange data on SparqNet
---

# OrbiterSDK <-> OrbiterSDK Data Bridging

Bridging data from a OrbiterSDK chain (A) to another (B) is simple:

* Chain A logs a request in the upcoming block and communicates this, along with the block reference, to the Gateway Network.

<figure><img src="../../.gitbook/assets/Diagram 1.0.png" alt=""><figcaption></figcaption></figure>

* A set of randomly selected Validators and Sentinels review Chain A's request and forward it to Chain B.

<figure><img src="../../.gitbook/assets/Diagram 2.png" alt=""><figcaption></figcaption></figure>

* Chain B retrieves the data from its chain, encapsulating it within a merkled item. This data and its internal reference are subsequently transmitted to the Gateway Network for permanent storage.

<figure><img src="../../.gitbook/assets/Diagram 3.png" alt=""><figcaption></figcaption></figure>

* Validators and Sentinels corroborate the data submitted by Chain B with other nodes from Chain B to validate its presence in the designated block.

<figure><img src="../../.gitbook/assets/Diagram 4.png" alt=""><figcaption></figcaption></figure>

* Validators and Sentinels signs the data and publish it inside the gateway, while also relaying it back to A.

<figure><img src="../../.gitbook/assets/Diagram 5.png" alt=""><figcaption></figcaption></figure>

* A verifies the signatures and checks if the randomly selected nodes were using the network's random number generator seed.

It is possible for Subnet A to only send information to the target chain without having to wait for an answer, this call can possibly trigger logic within the target chain, depending on how their developers decided to handle your message.

