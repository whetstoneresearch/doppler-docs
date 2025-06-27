---
icon: wind
---

# Airlock & modules

Airlock is what we refer to Doppler's implementation of a smart contract protocol facilitating the deployment of new tokens using a modular approach. Here's an overview of the components:

## Architecture

Different types of modules can be used to cover the several aspects of the token lifecycle.&#x20;

At a high level these can be understood as:&#x20;

<table><thead><tr><th width="243">Module Type</th><th>Role</th></tr></thead><tbody><tr><td>TokenFactories</td><td>Deploys the tokens</td></tr><tr><td>Bundlers</td><td>Enables purchasing at the time of creation, reducing MEV</td></tr><tr><td>PoolInitializers</td><td>Initializes a liquidity pool, for example on Uniswap V3</td></tr><tr><td>LiquidityMigrators</td><td>Migrates liquidity from one pool to another</td></tr><tr><td>GovernanceFactories</td><td>Deploys governance and timelock contracts, if configured</td></tr></tbody></table>

_Note: a "module" must be whitelisted before it can be used. If you are building a custom module, please get in touch with the Whetstone Research team._

{% hint style="info" %}
Refer to [GitHub](https://github.com/whetstoneresearch/doppler/blob/main/src/Airlock.sol) to view the open source Doppler Airlock implementation
{% endhint %}
