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

## How it works: in a bit more depth ...

1. When you create a token on an application like Pure Markets, you send a message to the Doppler contracts to create the token.&#x20;
2. The Doppler contracts create an ERC-20, a Uniswap v3 pool, a Uniswap v2 pool, and a Timelock. The code that facilitates this entire process is known as the "Doppler Airlock"
   1. Each one of these pieces is created by a “module”, which is an interface into the Doppler Airlock to facilitate individual trading actions like the liquidity bootstrapping pool, token factory, migrator (moves to generalized AMM), or timelock.
3. A share of the tokens set by the Interface are sent immediately to the Uniswap v3 pool
   1. The Uniswap v3 migrator contracts are designed to mitigate sniping by adjusting how liquidity is placed on the curve. There is no longer just one positions
   2. The max share of the entire token supply that can be sent is 50% and the minimum is 5% of the token supply. Whatever amount to send is determined by the Interface.
   3. There is a poke function that migrates the tokens to the next step.
      1. In the Uniswap v4 implementation, this is automatic, and we have other methodologies to make it automatic in the Uniswap v3 implementation.
   4. Once a certain price is met, the asset token (like ETH/USDC/USDT) are sent back to the Airlock, which calls the migrator module
4. Once receiving ETH or whatever token, the Airlock calls the migrator module. The module finds the most amount of LP tokens possible at the final price of the v3 pool. The migrator then utilizes whatever share of the tokens in the Airlock that it can, before sending the rest to the Timelock.
   1. The Uniswap v2 shares are held by the Timelock, meaning that the share sold to the open market is not lost forever when burned.
   2. The Timelock is gated for 3 months to ensure that users have time to trust the LP is locked
5. Bundled shares vest automatically, as does the integrator share and the Doppler share.

{% hint style="info" %}
If you're an onchain auction enjoyooor, we recommend reading the [Doppler Whitepaper](https://github.com/whetstoneresearch/docs/blob/main/whitepapers/doppler/Dutch_auction_Dynamic_Bonding_Curves.pdf) :point\_left:
{% endhint %}

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

