---
icon: wind
---

# Airlock & modules

Airlock is what we refer to Doppler's implementation of a smart contract protocol facilitating the deployment of new tokens using a modular approach. Here's an overview of the components:&#x20;

### Architecture

Different types of modules can be used to cover the several aspects of the token lifecycle:

| Module            | Role                                                    |
| ----------------- | ------------------------------------------------------- |
| TokenFactory      | Deploys the tokens                                      |
| GovernanceFactory | Deploys governance and timelock contracts               |
| PoolInitializer   | Initializes a liquidity pool, for example on Uniswap V3 |
| LiquidityMigrator | Migrates liquidity from one pool to another             |

_Note: a "module" must be whitelisted before it can be used._

{% hint style="info" %}
Refer to [GitHub](https://github.com/whetstoneresearch/doppler/blob/main/src/Airlock.sol) to view the open source Doppler Airlock implementation&#x20;
{% endhint %}

