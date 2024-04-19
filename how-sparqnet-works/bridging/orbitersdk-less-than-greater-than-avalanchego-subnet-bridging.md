---
description: How OrbiterSDK subnets interact with AvalancheGo subnets.
---

# OrbiterSDK <--> AvalancheGo Subnet Bridging

The Gateway Network is designed as an Avalanche Subnet, making it inherently compatible with the Avalanche Warp Messaging system. This compatibility means that blockchains developed with OrbiterSDK with AvalancheGo can easily leverage this warp system. They can efficiently communicate with other parts of the Avalanche network without relying entirely on the Gateway Network, enhancing speed and reliability.

By integrating the warp system into our bridging tactics, we enhance the reach of OrbiterSDK. It's not confined to just communicating with other OrbiterSDK blockchains but can connect with any subnet in the Avalanche framework.

In essence, any network—be it an Avalanche subnet or an OrbiterSDK subnet—can interface with one another. The mode of communication shifts slightly, with the warp messaging system facilitating exchanges from the Gateway Network to the Avalanche subnet.

See the example below on how messages are sent from an OrbiterSDK subnet to an AvalancheGo subnet, while the mechanisms to ensure data existence remains the same.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>Example of interoperability between Gateway Network and Avalanche</p></figcaption></figure>
