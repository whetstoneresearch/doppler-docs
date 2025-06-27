---
icon: square-terminal
---

# Implementation

Doppler has both a Uniswap v3 and Uniswap v4 implementation with differing features depending on your use cases.

### Which to use?

Doppler v3 uses static bonding curves to issue assets at incredibly low market capitalizations, eg. $50 or less. Doppler v3 is great for filtering through high volume content and assessing what may or may not be valuable, especially for use cases that the assets being created do not have any prior pricing history or market comparisons.

Doppler v4 uses dynamic bonding curves based on an onchain dutch auction, implemented in a Uniswap v4 hook. Doppler v4 is great for assets that have some sembalance of a known price that can be used to set the upper (max proceeds) and lower bounds (min proceeds) of the dutch auction.

Here's a bit more about how this works...

### Doppler v3 explainer

1. Applications send a message to the Doppler smart contracts to create a token. This message can be formed entirely utilizing the [doppler-sdk](../v3-sdk/factory.md)
2. The Doppler contracts create an ERC-20, a Uniswap v3 pool, a Uniswap v2 pool, and a Timelock. The code that facilitates this entire process is known as the ["Doppler Airlock"](airlock-and-modules.md)
   1. Each one of these pieces is created by a “module”, which is an interface into the Doppler Airlock to facilitate individual trading actions like the liquidity bootstrapping pool, token factory, migrator (moves to generalized AMM), or timelock.
3. A share of the tokens set by the Interface are sent immediately to the Uniswap v3 pool
   1. The Uniswap v3 migrator contracts are designed to mitigate sniping by adjusting how liquidity is placed on the curve. There is no longer just one position.
      1. The multiple positions determine the steepness of the curve. The steeper the curve, the more tokens are sold near the migration price of the curve. The modulation of this curve via market forces is supported in the Uniswap v4.
   2. The max share of the entire token supply that can be sent is 50% and the minimum is 5% of the token supply. Whatever amount to send is determined by the Interface.
   3. There is a poke function that migrates the tokens to the next step.
      1. In the Uniswap v4 implementation, this is automatic, and we have other methodologies to make it automatic in the Uniswap v3 implementation.
   4. Once a certain price is met, the asset token (like ETH/USDC/USDT) are sent back to the Airlock, which calls the migrator module
4. Once receiving ETH or whatever pre-specified token, the Airlock calls the migrator module. The module finds the most amount of LP tokens possible at the final price of the v3 pool. The migrator then utilizes whatever share of the tokens in the Airlock that it can, before sending the rest to the Timelock.
   1. The Uniswap v2 shares are held by the Timelock, meaning that the share sold to the open market is not lost forever when burned.
   2. The Timelock is gated for a time period of N, eg. 3 months, to ensure that users have time to trust the LP is locked
5. Bundled shares vest automatically, as does the integrator share and the Doppler share. Additional vesting can be defined as a percentage of inflation relative to total token supply.

{% hint style="info" %}
If you're an onchain auction enjoyooor, we recommend reading the [Doppler Whitepaper](https://github.com/whetstoneresearch/docs/blob/main/whitepapers/doppler/Dutch_auction_Dynamic_Bonding_Curves.pdf) :point\_left:
{% endhint %}

### Doppler v4 explainer

{% hint style="info" %}
The v4 implementation is now available in beta. See [contract addresses](../resources/contract-addresses.md) for supported deployments. Please reach out to the Whetstone Research team before implementing in production due to potential complexity and/or edge cases that must be accounted for.
{% endhint %}

The Uniswap v4 version of Doppler is exactly the same as the Uniswap v3 version for all the features mentioned above **EXCEPT** the liquidity bootstrapping step.

{% hint style="info" %}
**Module Combinations**: Both Doppler V3 and V4 pools can use fee streaming functionality. Simply combine any pool initializer (V3 or V4) with the UniswapV4Migrator to enable fee streaming when migrating to Uniswap V4. The modular architecture allows flexible combinations:

* V3 Initializer + V2 Migrator = Doppler V3 pool → Uniswap V2 pool
* V3 Initializer + V4 Migrator = Doppler V3 pool → Uniswap V4 pool with fee streaming
* V4 Initializer + V4 Migrator = Doppler V4 pool → Uniswap V4 pool with fee streaming
{% endhint %}

#### Uniswap v4 Permissions

The Doppler Protocol makes use of 4 Uniswap v4 hook functions in its contract:

* `afterInitialize`
  * Used to place initial liquidity positions
* `beforeSwap`
  * Used to trigger rebalancing of the bonding curve if the Protocol has not yet rebalanced in the current epoch
* `afterSwap`
  * Used to account the total amount of asset tokens sold, `totalTokensSold`, and the total amount of numeraire tokens received from asset sales, `totalProceeds`
  * The Protocol excludes the swap fee, consisting of the LP fee and the Uniswap Protocol fee, from the accounted amounts such that it doesn't reinvest LP fees or attempt to reinvest protocol fees taken by Uniswap v4
* `beforeAddLiquidity`
  * Used to trigger a revert if a user attempts to provide liquidity. This is necessary because the Protocol doesn't want any external liquidity providers beside itself.
    * External liquidity providers are blocked because it may result in a situation where the Doppler Protocol is unable to reset the underlying Uniswap v4 pool's accounting, creating accounting issues for the Protocol.

As previously stated, the main difference of v4-Doppler vs. v3-Doppler is the modulation of the positions placed is dynamic when compared to the Uniswap v3 version. This is the dynamic part referred to in "dynamic bonding curves".

### Dynamic Dutch-auctions

Curve Accumulation

The Doppler Protocol rebalance its bonding curve according to token sales along a pre-defined schedule based on the number of tokens to sell, `numTokensToSell`, over the duration, `endingTime - startingTime`. This rebalance occurs immediately preceeding the first swap in every epoch, in the `beforeSwap` hook. If the hook doesn't have any swaps in a given epoch then the rebalance applies retroactively to all missed epochs.

#### Max Dutch Auction

If sales are behind schedule, the curve is reduced via a dutch auction mechanism according to the relative amount that the Protocol is behind schedule. The maximum amount to dutch auction the curve in a single epoch is computed as the `endingTick - startingTick` divided by the total number of epochs, `(endingTime - startingTime) / epochLength`. In the case that there was a net sold amount of zero or less, computed as `totalTokensSold - totalTokensSoldLastEpoch`, the Protocol utilizes a dutch auction to shift the curve by this maximum amount.

#### Relative Dutch Auction

If the net sold amount is greater than zero, but the Protocol hasn't sold as many tokens as expected, computed as `percentage(elapsed time / duration) * numTokensToSell`, then the Protocol executes a dutch auction of the curve by the relative amount that it is undersold by multiplied by the maximum dutch auction amount, e.g. if the Protocol sold 80% of the expected amount, it undersold by 20% and thus will dutch auction by 20% of the maximum dutch auction amount.

#### Oversold Case

If sales are ahead of schedule, i.e. `totalTokensSold` is greater than the expected amount sold, computed as `percentage(elapsed time / duration) * numTokensToSell`, the Protocol moves the curve upwards by the amount that it has oversold by. The Protocol computes this increase as the delta between the current tick and the expected tick, which is generally the upper tick of the upper slug, which represents the point at which the Protocol has sold the expected amount (See Liquidity Placement).

#### `tickAccumulator`

For whichever of the above outcomes the Protocol has hit, it accumulates a tick delta to the `tickAccumulator`. This value is used to derive the current bonding curve at any given time. It is derived by the lowermost tick of the curve, `tickLower`, as the `startingTick + tickAccumulator`. Additionally, it derives the uppermost tick of the curve, `tickUpper`, as the `tickLower + gamma`. We can see how the `tickAccumulator` is accumulated in this [graph](https://www.desmos.com/calculator/fjnd0mcpst), with the red line corresponding to the max dutch auction case, the orange line corresponding to the relative dutch auction case, and the green line corresponding to the oversold case.

### Terminology in Dynamic Bonding Curves (Slugs)

Within the bonding curve, the Protocol places 3 different types of liquidity positions, aka slugs:

* Lower slug
  * Positioned below the current price, allowing for all purchased asset tokens to be sold back into the curve
* Upper slug
  * Positioned above the current price, allowing for asset tokens to be purchased, places enough tokens to reach the expected amount of tokens sold
* Price discovery slug(s)
  * Positioned above the upper slug, places enough tokens in each slug to reach the expected amount sold in the next epoch
  * Hook creators can pick an arbitrary amount of price discovery slugs, up to a maximum amount

#### Lower Slug

The lower slug is generally placed ranging from the global tickLower to the current tick. The Protocol places the total amount of proceeds from asset sales, `totalProceeds`, into the slug, allowing the users to sell their tokens back into the curve. The lower slug must have enough liquidity to support all tokens being sold back into the curve.

Ocassionally, the Protocol may not have sufficient `totalProceeds` to support all tokens being sold back into the curve with the usual slug placement. In this case, it computes the average clearing price of the tokens, computed as `totalProceeds / totalTokensSold` and place the slug at the tick corresponding to that price with a minimally sized range, i.e. range size of `tickSpacing`.

#### Upper Slug

The upper slug is generally placed between the current tick and a delta, computed as `epochLength / duration * gamma`. The Protocol supplies the delta between the expected amount of tokens sold, computed as `percentage(elapsed time / duration) * numTokensToSell`, and the actual `totalTokensSold`. In the case that `totalTokensSold` is greater than the expected amount of tokens sold, it skips the slug and instead simply set the ticks in storage both as the current tick.

#### Price Discovery Slugs

The price discovery slugs are generally placed between the upper slug upper tick and the top the bonding curve, `tickUpper`. The hook creator determines at the time of deployment how many price discovery slugs should be placed. The Protocol places the slugs equidistant between the upper slug's upper tick and the `tickUpper`, contiguously. the Protocol supplies tokens in each slug according to the percentage time difference between epochs multiplied by the `numTokensToSell`. Since the Protocol is supplying amounts according to remaining epochs, if it runs out of future epochs to supply for, it stops placing slugs. In the last epoch there will be no price disovery slugs.
