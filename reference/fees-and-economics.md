---
icon: hand-holding-dollar
---

# Fees & economics

Doppler's smart contracts contain enshrined an in-protocol, enshrined fee system.&#x20;

Every time a token is created or swapped, fees are calculated & paid for by the users of Doppler.&#x20;

These fee's are paid for in $ETH.&#x20;



*   Interface Fee&#x20;

    * This fee is configurable from third party application interfaces
    * This can be set between [<mark style="color:red;">{to confirm: 0 - 25 basis points, or 0.025% of swaps}</mark>](#user-content-fn-1)[^1]
    * See more in the [SDK reference](broken-reference)


* Contract Fee
  * This fee is fixed at <mark style="color:red;">{to confirm: 75 basis points, or 0.075% of swaps}</mark>
  * See more in the [Treasury](treasury.md) section



It is expected that eventually in subsequent versions of the protocol other tokens may be utilized in order to pay for the protocol fees. This may dependent on the application/interface's parameterization, the token creator's preference, the network the contracts are deployed to, or other tbd configurations.



[^1]: 
