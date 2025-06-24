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

## Fee Streaming (Doppler V4)

Doppler V4 introduces the **StreamableFeesLocker**, a new mechanism for distributing trading fees to multiple beneficiaries over time. This feature is exclusively available for tokens launched through Doppler V4.

### How it works

When launching a token on Doppler V4, creators can:

* **Standard Governance**: 90% of liquidity goes to a timelock contract, 10% goes to the StreamableFeesLocker
* **No-op Governance**: 100% of liquidity is permanently locked in the StreamableFeesLocker

The StreamableFeesLocker then distributes trading fees from the locked liquidity position to designated beneficiaries based on their configured share percentages.

### Key Features

* Multiple beneficiaries can receive ongoing fee distributions
* Beneficiaries can update their receiving address
* Anyone can trigger fee distributions (permissionless)
* Supports both temporary locks (standard governance) and permanent locks (no-op governance)

For detailed implementation examples, see the [StreamableFeesLocker guide](/doppler-v4-sdk-reference/streamable-fees-locker.md) and [Token Launch Examples](/doppler-v4-sdk-reference/token-launch-examples.md).
