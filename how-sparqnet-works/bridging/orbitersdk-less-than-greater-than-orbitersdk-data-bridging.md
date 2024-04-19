---
description: How native chains exchange data on SparqNet.
---

# OrbiterSDK <--> OrbiterSDK Data Bridging

Bridging arbitrary data from an OrbiterSDK chain (A) to another (B) is simple:

* Chain A sends a request in the upcoming block and communicates this, along with the block reference, to the Gateway Network. This request is written to the next block and relayed to SparqNet.

<figure><img src="../../.gitbook/assets/Diagram 1.0.png" alt=""><figcaption><p>A sends a request to the Gateway</p></figcaption></figure>

* A set of Validators and Sentinels are randomly selected using RandomGen (which works exactly the same way in all nodes in the network). The selected Validators and Sentinels review Chain A's request and forward it to Chain B.

<figure><img src="../../.gitbook/assets/Diagram 2.png" alt=""><figcaption><p>Validators and Sentinels in the Gateway relay A's request to B</p></figcaption></figure>

* Chain B receives the request from the Gateway and retrieves the data from its chain, encapsulating it within a merkled item. This data and its internal reference are subsequently transmitted back to the Gateway Network for permanent storage.

<figure><img src="../../.gitbook/assets/Diagram 3.png" alt=""><figcaption><p>B gathers data from itself and sends it back to Gateway</p></figcaption></figure>

* Validators and Sentinels then check and corroborate the data submitted by Chain B with other nodes from Chain B to validate its presence in the designated block.

<figure><img src="../../.gitbook/assets/Diagram 4.png" alt=""><figcaption><p>Gateway checks other B nodes to verify data</p></figcaption></figure>

* Once data is verified, Validators and Sentinels sign the data and publish it inside the Gateway, while also relaying it back to A along with their signatures.

<figure><img src="../../.gitbook/assets/Diagram 5.png" alt=""><figcaption><p>Gateway relays signed data back to A</p></figcaption></figure>

* A verifies the signatures and checks if the randomly selected nodes were using the network's RandomGen seed. If everything matches, the exchange is complete, but if not, there's a malicious node in the network which should be reported.

It is possible for Subnet A to only send information to the target chain without having to wait for an answer. This call can possibly trigger logic within the target chain, depending on how their developers decided to handle your message.

