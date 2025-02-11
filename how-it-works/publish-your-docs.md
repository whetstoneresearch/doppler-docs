---
icon: lightbulb
---

# Explainer

{% stepper %}
{% step %}
### Tokens get created

Any application can permissonlessly create tokens from the Doppler smart contracts

These tokens can be memecoins, RWAs, governance tokens — whatever your app focuses on
{% endstep %}

{% step %}
### Onchain dutch auction occurs

Tokens go through a price discovery period, fully onchain
{% endstep %}

{% step %}
### Dynamic bonding curve starts

Once a floor price is reached, tokens are available on a customized, dynamic bonding curve
{% endstep %}

{% step %}
### Liquidity moved to AMM

Once a specific value of tokens have been purchased or swapped, the tokens and LP positions automatically get migrated to Uniswap (v2 or v4), bootstrapping their initial liquidity&#x20;
{% endstep %}

{% step %}
### Complete :tada:

This newly created token is freely swappable across all Uniswap supported interfaces and able to be utilized throughout your favorite applications or Ethereum DeFi protocols
{% endstep %}
{% endstepper %}



### Design principles

* **Capital efficiency**&#x20;
  * Uniswap and Doppler are highly optimized to help tokens find their fair market price.&#x20;
* **MEV protecting**
  * Users are protected from sniping bots compared to other platforms due to the novel dynamics of the dutch auction based bonding curve, disrupting the dynamics of apeing in early.&#x20;
* **Programmability**
  * Doppler's smart contracts are fully onchain and composable with other programs.&#x20;
  * We envision a large ecosystem and variety of mechanisms for token distribution & management, such as: vesting, airdrops, incentives, dao management, and more.&#x20;
* **EVM & DeFI native**
  * Automatically integrate your tokens into the latest & greatest Ethereum based DeFi protocols.
* **Uniswap compatibility**
  * Tokens created on Doppler that reach their specific goals of liquidity will automatically get migrated to their own Uniswap v2 or Uniswap v4 pools and swappable from Uniswap supported interfaces.

