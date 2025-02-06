---
icon: lightbulb
---

# Explainer



{% stepper %}
{% step %}
### Tokens get created

Any application can permissonlessly create tokens from the Doppler smart contracts
{% endstep %}

{% step %}
### Onchain dutch auction occurs

Tokens go through a price discovery period, initially descending
{% endstep %}

{% step %}
### Bonding curve starts

Once a floor price is reached, tokens available on a customized bonding curve
{% endstep %}

{% step %}
### Liquidity moved to AMM

Once a specific value of tokens have been purchased or swapped, the tokens and LP positions automatically get migrated to Uniswap v2 or v4, bootstrapping their initial liquidity&#x20;
{% endstep %}

{% step %}
### Complete :tada:

This newly created token is freely swappable across all Uniswap supported interfaces
{% endstep %}
{% endstepper %}

### Design principles

* Capital efficiency&#x20;
  * Uniswap and Doppler are highly optimized to help tokens find their fair market price.&#x20;
* MEV protecting
  * Users are protected from sniping bots compared to other platforms due to the novel dynamics of the dutch auction based bonding curve, disrupting the dynamics of apeing in early.&#x20;
* Programmability
  * Doppler's smart contracts are fully onchain and composable with other programs.&#x20;
* Uniswap compatability
  * Tokens created on Doppler that reach their specific goals of liquidity will automatically get migrated to their own Uniswap v2 or Uniswap v4 pools and swappable from Uniswap supported interfaces.
* Ecosystem-first&#x20;
  * Whetstone envisions a rich ecosystem of downstream integrations where tokens created on Doppler can integrate with a variety of mechanisms for token distribution & management, such as: vesting, airdrops, incentives, dao management, and more. &#x20;



{% hint style="info" %}
If you'd like to dive deeper, we recommend reading the [Doppler Whitepaper](https://github.com/whetstoneresearch/docs/blob/main/whitepapers/doppler/Dutch_auction_Dynamic_Bonding_Curves.pdf) :point\_left:
{% endhint %}

