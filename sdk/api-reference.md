---
icon: terminal
---

# API reference

## Builder API Reference

Builders assemble type‑safe parameter objects for `DopplerFactory.createStaticAuction`, `DopplerFactory.createDynamicAuction`, and `DopplerFactory.createMulticurve`.

* Static auctions: Uniswap V3 style, fixed price range liquidity bootstrapping
* Dynamic auctions: Uniswap V4 hook, dynamic Dutch auction with epoch steps
* Multicurve auctions: Uniswap V4 initializer with multiple curves

### Common Concepts

* Governance defaults to `noOp` on supported chains (all except Ink)
* Fee tiers and tick spacing: 100→1, 500→10, 3000→60, 10000→200

***

## StaticAuctionBuilder

Methods (chainable):

* `tokenConfig({ name, symbol, tokenURI, yearlyMintRate? })`
* `saleConfig({ initialSupply, numTokensToSell, numeraire })`
* `withMarketCapRange({ marketCap: { start, end }, numerairePrice, fee?, numPositions?, maxShareToBeSold? })`
  * `marketCap.start` and `marketCap.end` are fully diluted market caps in USD (or whatever unit your numeraire is priced in)
  * Requires `saleConfig()` first
* `poolByTicks({ startTick, endTick, fee?, numPositions?, maxShareToBeSold? })`
* `withVesting({ duration?, cliffDuration?, recipients?, amounts? })`
* `withGovernance({ type: 'default' | 'custom' | 'noOp' })`
* `withMigration(MigrationConfig)`
* `withUserAddress(address)`
* `build()` → `CreateStaticAuctionParams`

***

## DynamicAuctionBuilder

Methods (chainable):

* `tokenConfig({ name, symbol, tokenURI, yearlyMintRate? })`
* `saleConfig({ initialSupply, numTokensToSell, numeraire? })`
* `poolConfig({ fee, tickSpacing })`
* `withMarketCapRange({ marketCap: { start, end }, numerairePrice, minProceeds, maxProceeds, duration?, epochLength? })`
  * `marketCap.start` and `marketCap.end` are fully diluted market caps in USD (or whatever unit your numeraire is priced in)
  * Requires `saleConfig()` and `poolConfig()` first
* `auctionByTicks({ startTick, endTick, minProceeds, maxProceeds, duration?, epochLength?, gamma? })`
* `withVesting({ duration?, cliffDuration?, recipients?, amounts? })`
* `withGovernance({ type: 'default' | 'custom' | 'noOp' })`
* `withMigration(MigrationConfig)`
* `withUserAddress(address)`
* `withTime({ startTimeOffset?, blockTimestamp? })`
* `build()` → `CreateDynamicAuctionParams`

***

## MulticurveBuilder

Methods (chainable):

* `tokenConfig({ name, symbol, tokenURI, yearlyMintRate? })`
* `saleConfig({ initialSupply, numTokensToSell, numeraire })`
* `withCurves({ numerairePrice, curves, fee?, tickSpacing?, beneficiaries? })`
  * Requires `saleConfig()` first
  * `curves`: Array of `{ marketCap: { start, end }, numPositions, shares }`
  * `marketCap.start` and `marketCap.end` are fully diluted market caps in USD (or whatever unit your numeraire is priced in)
  * Shares must sum to 1e18 (100%)
* `poolConfig({ fee, tickSpacing, curves, beneficiaries? })`
  * `curves`: Array of `{ tickLower, tickUpper, numPositions, shares }`
* `withVesting({ duration?, cliffDuration?, recipients?, amounts? })`
* `withGovernance({ type: 'default' | 'custom' | 'noOp' })`
* `withMigration(MigrationConfig)`
* `withUserAddress(address)`
* `build()` → `CreateMulticurveParams`

### Multicurve rules

* First curve's `marketCap.start` = the launch price
* Curves must be contiguous or overlapping (no gaps)
* Shares must sum to exactly 1e18 (100%)

***

## Factory methods

```ts
const { poolAddress, tokenAddress } = await sdk.factory.createStaticAuction(params)
const { hookAddress, tokenAddress, poolId } = await sdk.factory.createDynamicAuction(params)
const { poolAddress, tokenAddress } = await sdk.factory.createMulticurve(params)
```
