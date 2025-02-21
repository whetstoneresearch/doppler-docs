---
icon: wind
---

# Airlock & modules

Airlock is what we refer to Doppler's implementation of a smart contract protocol facilitating the deployment of new tokens using a modular approach. Here's an overview of the components:&#x20;

## Architecture

Different types of modules can be used to cover the several aspects of the token lifecycle:

<table><thead><tr><th width="243">Module</th><th>Role</th></tr></thead><tbody><tr><td>TokenFactory</td><td>Deploys the tokens</td></tr><tr><td>GovernanceFactory</td><td>Deploys governance and timelock contracts</td></tr><tr><td>PoolInitializer</td><td>Initializes a liquidity pool, for example on Uniswap V3</td></tr><tr><td>LiquidityMigrator</td><td>Migrates liquidity from one pool to another</td></tr></tbody></table>

_Note: a "module" must be whitelisted before it can be used._

{% hint style="info" %}
Refer to [GitHub](https://github.com/whetstoneresearch/doppler/blob/main/src/Airlock.sol) to view the open source Doppler Airlock implementation
{% endhint %}
