---
icon: lightbulb
---

# Explainer

{% stepper %}
{% step %}
#### Tokens get created

Any application can permissonlessly create tokens by passing cusstomized inputs into the Doppler smart contracts. These tokens can be memecoins, RWAs, governance tokens - whatever your specific application uniquely focuses on.
{% endstep %}

{% step %}
#### Price discovery

Tokens go through a price discovery period, fully onchain, bootstrapping their own liquidity.
{% endstep %}

{% step %}
#### Migration

Once the parameterized amount of tokens have been purchased or swapped, the tokens and LP positions can optionally get migrated to Uniswap (v2, to grow liquidity overtime through fees), or to Uniswap v4 (with support for custom fees), or triggering other contract interactions.
{% endstep %}

{% step %}
#### Treasury management & governance

Tokens can optionally be created with OpenZeppelin Governor Treasuries to give communities a  home for their ecosystem and way to propose and vote on actions as a collective.
{% endstep %}

{% step %}
#### Complete :tada:

This newly created token is freely swappable across all Uniswap supported interfaces and able to be utilized throughout your favorite applications, DeFi protocols, and interfaces.
{% endstep %}
{% endstepper %}

### Design principles

* **Capital efficiency**
  * Uniswap and Doppler are highly optimized to help tokens find their fair market price.
* **MEV protecting**
  * Users are protected from sniping bots compared to other platforms due to the novel dynamics of the dutch auction based bonding curve, disrupting the dynamics of apeing in early.
* **Programmability**
  * Doppler's smart contracts are fully onchain and composable with other programs.
  * We envision a large ecosystem and variety of mechanisms for token distribution & management, such as: vesting, airdrops, incentives, dao management, and more.
* **Optionality**
  * Turn on and off different modules, eg. use governance, or opt out of it entirely.
* **EVM & DeFI native**
  * Automatically integrate your tokens into the latest & greatest Ethereum based DeFi protocols.
* **Uniswap compatibility**
  * Tokens created on Doppler that reach their specific goals of liquidity will automatically get migrated to their own Uniswap v2 or Uniswap v4 pools and swappable from Uniswap supported interfaces. Eventually other AMMs may be supported.
* **Custom fees**
  * Applications can customize multiple benefeciary addresses with different fee tiers, enabling a flexible model that better fits into a businesses and their customer's unique goals.&#x20;
