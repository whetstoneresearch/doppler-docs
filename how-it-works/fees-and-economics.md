---
icon: hand-holding-dollar
---

# Fees & economics

Doppler's smart contracts contain enshrined an in-protocol, enshrined fee system.&#x20;

Every time a token is created or swapped, fees are calculated & paid for by the users of Doppler.&#x20;

These fee's are paid for in $ETH.&#x20;

* Integrator Fee - up to 95% of LP fees
  * This fee is configurable by third party application integrators&#x20;
    * This can be set between 0 - 95% of total LP fees earned
    * See more in the [SDK reference](/doppler-v3-sdk-reference/factory.md)
*   Protocol Fee - 5% of LP fees
    * This fee is fixed at 5% of total LP fees earned

It is expected that eventually in subsequent versions of the protocol other tokens may be utilized in order to pay for the protocol fees. This may dependent on the application/interface's parameterization, the token creator's preference, the network the contracts are deployed to, or other tbd configurations.
