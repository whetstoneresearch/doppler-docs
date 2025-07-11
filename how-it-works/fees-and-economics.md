---
icon: hand-holding-dollar
---

# Fees & economics

Doppler's smart contracts contain enshrined an in-protocol, enshrined fee system.

Every time a token is created or swapped, fees are calculated & paid for by the users of Doppler.

These fee's are paid for in $ETH.

* Integrator Fee - up to 95% of LP fees
  * This fee is configurable by third party application integrators
    * This can be set between 0 - 95% of total LP fees earned
    * See more in the [SDK reference](../v3-sdk/factory.md)
* Protocol Fee - 5% of LP fees
  * This fee is fixed at 5% of total LP fees earned

It is expected that eventually in subsequent versions of the protocol other tokens may be utilized in order to pay for the protocol fees. This may dependent on the application/interface's parameterization, the token creator's preference, the network the contracts are deployed to, or other tbd configurations.

## Custom fees&#x20;

The **StreamableFeesLocker** is a new mechanism for distributing trading fees to multiple beneficiaries over time. When used with the Doppler v4 migrator, can enable _customizable application specific fees_.

### Key Features

* Multiple beneficiaries can receive ongoing fee distributions
* Fee tiers can be customized during token creation, eg. 0.1%, 1%, or 10%
* Beneficiaries can update their receiving address
* Anyone can trigger fee distributions (permissionless)
* Supports both temporary locks (standard governance) and permanent locks (no-op governance)
* **Compatible with both Doppler V3 and V4 pools**&#x20;
  * Any pool type can use fee streaming by selecting the UniswapV4Migrator

### Key Concepts

### Beneficiaries

Beneficiaries are addresses that receive a share of trading fees from the locked liquidity.&#x20;

Each beneficiary has:

* **Address**: The recipient address for fee claims
* **Shares**: The proportion of fees they receive (in WAD units, where 1e18 = 100%)

### Custom Fee Streaming

The StreamableFeesLocker smart contract:

* Holds 10% of migrated liquidity in a locked position
* Distributes trading fees to beneficiaries based on their shares
* Allows beneficiaries to claim accumulated fees over time
* Supports beneficiary address updates

### Pool Compatibility

Fee streaming is available for:

* **Doppler V3 pools**: Use `UniswapV3Initializer` (creates Doppler V3 pool) + `UniswapV4Migrator` (enables fee streaming)
* **Doppler V4 pools**: Use `UniswapV4Initializer` (creates Doppler V4 pool) + `UniswapV4Migrator` (enables fee streaming)

Both pool types migrate to Uniswap V4 and can leverage the StreamableFeesLocker for ongoing fee distribution. For detailed examples, see the [StreamableFeesLocker guide](../v4-sdk/custom-fees.md) and [Token Launch Examples](../v4-sdk/examples.md).
