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
* `withMarketCapRange({ marketCap: { start, min }, numerairePrice, minProceeds, maxProceeds, duration?, epochLength? })`
  * `marketCap.start` is the starting market cap (auction begins here), `marketCap.min` is the floor price the auction descends to
  * Both values are fully diluted market caps in USD (or whatever unit your numeraire is priced in)
  * Requires `saleConfig()` first (do NOT use `poolConfig()` with this method - they are mutually exclusive)
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

## RehypeDopplerHook Configuration

The `withRehypeDopplerHook()` method on `MulticurveBuilder` enables advanced fee distribution and buyback mechanisms.

Methods (chainable on MulticurveBuilder):

* `withRehypeDopplerHook({ hookAddress, buybackDestination, customFee, assetBuybackPercentWad, numeraireBuybackPercentWad, beneficiaryPercentWad, lpPercentWad, graduationCalldata? })`
* `withDopplerHookInitializer(address)` — required when using Rehype
* `withNoOpMigrator(address)` — required (Rehype pools don't migrate)

### RehypeDopplerHookConfig parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `hookAddress` | `Address` | Deployed RehypeDopplerHook (must be whitelisted) |
| `buybackDestination` | `Address` | Receives bought-back tokens |
| `customFee` | `number` | Swap fee in bps (3000 = 0.3%) |
| `assetBuybackPercentWad` | `bigint` | % for asset buyback (WAD, 1e18 = 100%) |
| `numeraireBuybackPercentWad` | `bigint` | % for numeraire buyback (WAD) |
| `beneficiaryPercentWad` | `bigint` | % for beneficiaries (WAD) |
| `lpPercentWad` | `bigint` | % for LPs (WAD) |
| `graduationCalldata` | `Hex` | Optional calldata executed on graduation |

### Rehype rules

* All four percentage parameters must sum to exactly `WAD` (1e18)
* `hookAddress` must be enabled in `DopplerHookInitializer`
* Use `noOp` migration — Rehype pools enter "Locked" status
* Beneficiary shares must sum to `WAD`; protocol owner requires minimum 5%

***

## Rehype Pool Methods

After creating a Rehype pool, interact with it via the SDK:

```ts
const pool = await sdk.getMulticurvePool(assetAddress);

// Collect fees (anyone can call; distributes automatically)
const { fees0, fees1, transactionHash } = await pool.collectFees();

// Get current fee distribution info
const feeInfo = await pool.getFeeDistributionInfo();
// Returns: { assetBuybackPercentWad, numeraireBuybackPercentWad, beneficiaryPercentWad, lpPercentWad }

// Get accumulated hook fees
const hookFees = await pool.getHookFees();
// Returns: { fees0, fees1, beneficiaryFees0, beneficiaryFees1, airlockOwnerFees0, airlockOwnerFees1 }
```

### Fee claiming (on-chain)

| Method | Caller | Description |
|--------|--------|-------------|
| `collectFees(asset)` | Anyone | Transfers beneficiary fees to `buybackDst` |
| `claimAirlockOwnerFees(asset)` | Airlock owner only | Claims 5% protocol fee |
| `setFeeDistribution(poolId, ...)` | `buybackDst` only | Updates fee distribution |

***

## Factory methods

```ts
const { poolAddress, tokenAddress } = await sdk.factory.createStaticAuction(params)
const { hookAddress, tokenAddress, poolId } = await sdk.factory.createDynamicAuction(params)
const { poolAddress, tokenAddress } = await sdk.factory.createMulticurve(params)
```
