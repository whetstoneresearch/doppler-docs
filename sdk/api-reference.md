---
icon: terminal
---

# API reference

## Builder API Reference

This document specifies the fluent builder APIs used to create Doppler auctions. Builders assemble type‑safe parameter objects for `DopplerFactory.createStaticAuction`, `DopplerFactory.createDynamicAuction`, and `DopplerFactory.createMulticurve`, applying sensible defaults and computing derived values (ticks, gamma) where helpful.

* Static auctions: Uniswap V3 style, fixed price range liquidity bootstrapping
* Dynamic auctions: Uniswap V4 hook, dynamic Dutch auction with epoch steps
* Multicurve auctions: Uniswap V4 initializer with multiple curves

All types referenced are exported from `src/types.ts`.

### Common Concepts

* Token specification:
  * `standard` (default): DERC20 with optional vesting and yearly mint rate
* Governance is required:
  * Call `withGovernance(...)` in all cases.
  * `withGovernance({ type: 'default' })` applies standard governance defaults.
  * `withGovernance({ type: 'noOp' })` explicitly selects no‑op governance (requires chain support).
  * Or use `{ type: 'custom', initialVotingDelay, initialVotingPeriod, initialProposalThreshold }` to customize.
* Fee tiers and tick spacing: 100→1, 500→10, 3000→60, 10000→200

***

## Market Cap Configuration (Recommended)

The `withMarketCapRange()` and `withCurves()` methods provide a friendly API for configuring auctions. Instead of dealing with abstract tick values, you specify:

* **Target market cap range** in USD (e.g., $100k to $10M)
* **Numeraire price** in USD (e.g., ETH = $3000)

The SDK automatically converts these values to the appropriate tick ranges.

***

## StaticAuctionBuilder (V3‑style)

Recommended for fixed price range launches with Uniswap V3.

Methods (chainable):

* tokenConfig(params)
  * Standard: `{ name, symbol, tokenURI, yearlyMintRate? }`
    * Defaults: `yearlyMintRate = DEFAULT_V3_YEARLY_MINT_RATE (0.02e18)`
* saleConfig({ initialSupply, numTokensToSell, numeraire })
* Price specification methods (use one, not multiple):
  * **withMarketCapRange({ marketCap, numerairePrice, ... })** Recommended
  * poolByTicks({ startTick?, endTick?, fee?, numPositions?, maxShareToBeSold? })
    * Defaults: `fee = DEFAULT_V3_FEE (10000)`, `startTick = DEFAULT_V3_START_TICK`, `endTick = DEFAULT_V3_END_TICK`, `numPositions = DEFAULT_V3_NUM_POSITIONS`, `maxShareToBeSold = DEFAULT_V3_MAX_SHARE_TO_BE_SOLD`
* withVesting({ duration?, cliffDuration?, recipients?, amounts? } | undefined)
  * Omit to disable vesting. Default duration if provided but undefined is `DEFAULT_V3_VESTING_DURATION`.
  * `recipients`: Optional array of addresses to receive vested tokens
  * `amounts`: Optional array of token amounts per recipient
* withGovernance(GovernanceConfig)
* withMigration(MigrationConfig)
* withUserAddress(address)
* withIntegrator(address?)
  * Defaults to zero address if omitted
* build(): CreateStaticAuctionParams
  * Throws if required sections are missing

### Market Cap Config Interface (Static)

```typescript
interface StaticAuctionMarketCapConfig {
  marketCap: { start: number; end: number };  // USD market cap range
  numerairePrice: number;                      // Numeraire price in USD
  tokenSupply?: bigint;                        // Override (defaults to initialSupply)
  tokenDecimals?: number;                      // Default: 18
  numeraireDecimals?: number;                  // Default: 18
  fee?: number;                                // Default: 10000 (1%)
  numPositions?: number;                       // Default: 15
  maxShareToBeSold?: bigint;                   // Default: 0.35e18 (35%)
}
```

### Example (Market Cap)

```ts
const params = sdk.buildStaticAuction()
  .tokenConfig({ name: 'My Token', symbol: 'MTK', tokenURI: 'https://example.com/mtk.json' })
  .saleConfig({ initialSupply: parseEther('1_000_000_000'), numTokensToSell: parseEther('500_000_000'), numeraire: WETH })
  .withMarketCapRange({
    marketCap: { start: 100_000, end: 10_000_000 }, // $100k to $10M fully diluted
    numerairePrice: 3000, // ETH = $3000 USD
  })
  .withVesting({ duration: BigInt(365*24*60*60) })
  .withGovernance({ type: 'default' })
  .withMigration({ type: 'uniswapV2' })
  .withUserAddress(user)
  .build()
```

***

## DynamicAuctionBuilder (V4‑style)

Recommended for Dynamic Dutch Auctions where price moves over epochs using Uniswap V4 hooks.

Methods (chainable):

* tokenConfig(params)
  * Standard: `{ name, symbol, tokenURI, yearlyMintRate? }`
    * Defaults: `yearlyMintRate = DEFAULT_V4_YEARLY_MINT_RATE (0.02e18)`
* saleConfig({ initialSupply, numTokensToSell, numeraire? })
  * Defaults: `numeraire = ZERO_ADDRESS` (token is paired against ETH)
* poolConfig({ fee, tickSpacing })
* Price configuration methods (use one, not multiple):
  * **withMarketCapRange({ marketCap, numerairePrice, minProceeds, maxProceeds, ... })** ⭐ Recommended
    * Configure via dollar-denominated market cap targets
    * Requires both `saleConfig()` AND `poolConfig()` to be called first
    * Auto-detects `tokenIsToken1` from numeraire address
    * Required: `marketCap: { start, end }`, `numerairePrice`, `minProceeds`, `maxProceeds`
    * Optional: `duration`, `epochLength`, `gamma`, `numPdSlugs`, `tokenDecimals`, `numeraireDecimals`
    * Defaults: `duration = 7 days`, `epochLength = 1 hour`, `numPdSlugs = 5`
  * auctionByTicks({ startTick, endTick, minProceeds, maxProceeds, duration?, epochLength?, gamma?, numPdSlugs? })
    * Defaults: `duration = DEFAULT_AUCTION_DURATION (604800)`, `epochLength = DEFAULT_EPOCH_LENGTH (3600)`, `numPdSlugs` optional
    * If `gamma` omitted, computed from ticks, duration, epoch length, and `tickSpacing`
  * auctionByPriceRange({ priceRange, minProceeds, maxProceeds, duration?, epochLength?, gamma?, tickSpacing?, numPdSlugs? })
    * Uses `pool.tickSpacing` unless `tickSpacing` is provided here
    * @deprecated: Use `withMarketCapRange()` instead
* withVesting({ duration?, cliffDuration?, recipients?, amounts? } | undefined)
  * Omit to disable vesting. Default duration if provided but undefined is `0` for dynamic auctions.
* withGovernance(GovernanceConfig)
  * Call is required
* withMigration(MigrationConfig)
* withUserAddress(address)
* withIntegrator(address?)
* withTime({ startTimeOffset?, blockTimestamp? } | undefined)
  * Controls auction time reference; if omitted, factory fetches latest block timestamp and uses 30s offset
* build(): CreateDynamicAuctionParams
  * Ensures `gamma` finalized, fills defaults, and throws if required sections are missing

### Market Cap Config Interface (Dynamic)

```typescript
interface DynamicAuctionMarketCapConfig {
  marketCap: { start: number; end: number };  // USD market cap range
  numerairePrice: number;                      // Numeraire price in USD
  minProceeds: bigint;                         // Required: minimum ETH to graduate
  maxProceeds: bigint;                         // Required: maximum ETH cap
  tokenSupply?: bigint;                        // Override (defaults to initialSupply)
  tokenDecimals?: number;                      // Default: 18
  numeraireDecimals?: number;                  // Default: 18
  duration?: number;                           // Default: 7 days (604800)
  epochLength?: number;                        // Default: 1 hour (3600)
  gamma?: number;                              // Auto-calculated if not provided
  numPdSlugs?: number;                         // Default: 5
}
```

### Example (Market Cap)

```ts
const params = sdk.buildDynamicAuction()
  .tokenConfig({ name: 'My Token', symbol: 'MTK', tokenURI: 'https://example.com/mtk.json' })
  .saleConfig({ initialSupply: parseEther('1_000_000_000'), numTokensToSell: parseEther('500_000_000'), numeraire: WETH })
  .poolConfig({ fee: 3000, tickSpacing: 60 }) // Required BEFORE withMarketCapRange!
  .withMarketCapRange({
    marketCap: { start: 500_000, end: 50_000_000 }, // $500k to $50M fully diluted
    numerairePrice: 3000, // ETH = $3000 USD
    minProceeds: parseEther('100'), // Min 100 ETH to graduate
    maxProceeds: parseEther('5000'), // Cap at 5000 ETH
  })
  .withGovernance({ type: 'default' })
  .withMigration({ type: 'uniswapV4', fee: 3000, tickSpacing: 60, streamableFees: { ... } })
  .withUserAddress(user)
  .build()
```

***

## MulticurveBuilder (V4)

Recommended when you want to seed a Uniswap V4 pool with multiple curves in a single initializer call. This supports richer liquidity distributions and works with any migration type (V2, V3, or V4).

Methods (chainable):

* tokenConfig(params)
  * Standard: `{ name, symbol, tokenURI, yearlyMintRate? }`
    * Defaults: `yearlyMintRate = DEFAULT_V4_YEARLY_MINT_RATE (0.02e18)`
* saleConfig({ initialSupply, numTokensToSell, numeraire })
* Curve configuration methods (use one, not multiple):
  * **withCurves({ numerairePrice, curves, ... })** ⭐ Recommended
    * Configure via dollar-denominated market cap ranges (no tick math required)
    * Requires `saleConfig()` to be called first
    * Auto-detects token ordering from numeraire address
    * `numerairePrice`: Price of numeraire in USD (e.g., 3000 for ETH at $3000)
    * `curves`: Array of `{ marketCap: { start, end }, numPositions, shares }`
    * Optional: `fee`, `tickSpacing`, `tokenDecimals`, `numeraireDecimals`, `beneficiaries`, `tokenSupply`
    * Shares must sum to exactly WAD (1e18 = 100%)
  * poolConfig({ fee, tickSpacing, curves, beneficiaries? })
    * Low-level tick-based configuration for advanced users
    * `curves`: Array of `{ tickLower, tickUpper, numPositions, shares }` where `shares` are WAD-based weights
* withVesting({ duration?, cliffDuration?, recipients?, amounts? } | undefined)
* withGovernance(GovernanceConfig)
  * Call is required
* withMigration(MigrationConfig)
  * Supports `uniswapV2`, `uniswapV3`, or `uniswapV4`
* withUserAddress(address)
* withIntegrator(address?)
* build(): CreateMulticurveParams

### Market Cap Curves Config Interface

```typescript
interface MulticurveMarketCapCurvesConfig {
  numerairePrice: number;                      // Numeraire price in USD (e.g., 3000)
  curves: MulticurveMarketCapRangeCurve[];     // Market cap-based curves
  tokenSupply?: bigint;                        // Override (defaults to initialSupply)
  tokenDecimals?: number;                      // Default: 18
  numeraireDecimals?: number;                  // Default: 18
  fee?: number;                                // Default: 500 (0.05%)
  tickSpacing?: number;                        // Derived from fee if not provided
  beneficiaries?: BeneficiaryData[];           // Optional fee streaming recipients
}

interface MulticurveMarketCapRangeCurve {
  marketCap: {
    start: number;       // Start market cap in USD (launch price for first curve)
    end: number;         // End market cap in USD
  };
  numPositions: number;  // Number of LP positions
  shares: bigint;        // WAD-based share (e.g., parseEther('0.3') = 30%)
}
```

### Multicurve Rules

* **First curve's `marketCap.start`** = the launch price
* **Curves must be contiguous or overlapping** (no gaps allowed)
* **Shares must sum to exactly 1e18 (100%)**
* Overlapping curves provide extra liquidity at key thresholds

### Example (Market Cap Curves)

```ts
const params = sdk.buildMulticurveAuction()
  .tokenConfig({ name: 'My Token', symbol: 'MTK', tokenURI: 'https://example.com/mtk.json' })
  .saleConfig({ initialSupply: parseEther('1_000_000_000'), numTokensToSell: parseEther('900_000_000'), numeraire: WETH })
  .withCurves({
    numerairePrice: 3000, // ETH = $3000 USD
    curves: [
      // Curve 1: Launch curve (concentrated liquidity at low market cap)
      { marketCap: { start: 500_000, end: 1_500_000 }, numPositions: 10, shares: parseEther('0.3') }, // 30%
      // Curve 2: Mid-range (provides depth as price rises)
      { marketCap: { start: 1_000_000, end: 5_000_000 }, numPositions: 15, shares: parseEther('0.4') }, // 40%
      // Curve 3: Upper range (moon bag for high market cap)
      { marketCap: { start: 4_000_000, end: 50_000_000 }, numPositions: 10, shares: parseEther('0.3') }, // 30%
    ],
  })
  .withVesting({ duration: BigInt(365*24*60*60) })
  .withGovernance({ type: 'default' })
  .withMigration({ type: 'uniswapV2' })
  .withUserAddress(user)
  .build()

const { poolAddress, tokenAddress } = await sdk.factory.createMulticurve(params)
```

***

## Method Call Order Requirements

{% hint style="warning" %}
`withMarketCapRange()` and `withCurves()` require certain methods to be called first.
{% endhint %}

### Static Auction
```
saleConfig() → withMarketCapRange()
```

`saleConfig()` provides:
- `numeraire` address (for token ordering auto-detection)
- `initialSupply` (for market cap calculations)

### Dynamic Auction
```
saleConfig() → poolConfig() → withMarketCapRange()
```

`poolConfig()` provides:
- `tickSpacing` (required for tick calculations)

### Multicurve
```
saleConfig() → withCurves()
```

Same as Static Auction - only `saleConfig()` required first.

***

## Validation & Error Handling

The SDK validates market cap configurations automatically:

### Errors (will throw)

* `saleConfig()` not called before `withMarketCapRange()` / `withCurves()`
* `poolConfig()` not called before `withMarketCapRange()` (Dynamic only)
* `start >= end` market cap
* Missing required fields (`minProceeds`/`maxProceeds` for Dynamic)
* Curve shares don't sum to exactly 1e18 (Multicurve)
* Gap between curves (Multicurve) - curves must be contiguous or overlapping

### Warnings (logged but allowed)

* Very small market caps (< $1000)
* Very large market caps (> $1B)
* Extreme price ratios (end/start > 10000x)

***

## Build Results

* Static: `CreateStaticAuctionParams` with fields: `token`, `sale`, `pool`, optional `vesting`, `governance`, `migration`, `integrator`, `userAddress`
* Dynamic: `CreateDynamicAuctionParams` with fields: `token`, `sale`, `auction`, `pool`, optional `vesting`, `governance`, `migration`, `integrator`, `userAddress`, optional `startTimeOffset`, optional `blockTimestamp`
* Multicurve: `CreateMulticurveParams` with fields: `token`, `sale`, `pool` (with `curves`), optional `vesting`, `governance`, `migration`, `integrator`, `userAddress`

Pass the built object directly to the factory:

```ts
const { poolAddress, tokenAddress } = await sdk.factory.createStaticAuction(staticParams)
const { hookAddress, tokenAddress: token2, poolId } = await sdk.factory.createDynamicAuction(dynamicParams)
const { poolAddress: pool3, tokenAddress: token3 } = await sdk.factory.createMulticurve(multicurveParams)
```

Notes:

* For doppler404 tokens, ensure `doppler404Factory` is configured on your target chain (see `src/addresses.ts`).
* Doppler404 tokenConfig supports optional `unit?: bigint` which defaults to `1000` when omitted.
* `integrator` defaults to zero address when omitted.
* `withTime` is only relevant to dynamic auctions.

***

## Comparison: Auction Types

| Aspect | Static (V3) | Dynamic (V4) | Multicurve (V4) |
|--------|-------------|--------------|-----------------|
| **Method** | `withMarketCapRange()` | `withMarketCapRange()` | `withCurves()` |
| **Protocol** | Uniswap V3 | Uniswap V4 Hook | Uniswap V4 Initializer |
| **Price movement** | Ascending bonding curve | Descending Dutch auction | Multiple overlapping curves |
| **Multiple curves** | No | No | Yes |
| **Proceeds config** | Not configurable | `minProceeds`, `maxProceeds` required | N/A |
| **Duration** | N/A | `duration`, `epochLength` | N/A |
| **Pre-requisites** | `saleConfig()` only | `saleConfig()` + `poolConfig()` | `saleConfig()` only |
| **Default fee** | 10000 (1%) | User-specified | 500 (0.05%) |
