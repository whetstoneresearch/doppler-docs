---
icon: terminal
---

# API reference

***

## DopplerSDK (EVM)

The top-level entry point for all EVM SDK operations. Import from `@whetstone-research/doppler-sdk`.

```ts
import { DopplerSDK } from '@whetstone-research/doppler-sdk'

const sdk = new DopplerSDK({
  publicClient,   // viem PublicClient
  walletClient,   // viem WalletClient (optional for read-only)
  chainId,        // SupportedChainId
})
```

### Builder shortcuts

```ts
sdk.buildStaticAuction()       // → StaticAuctionBuilder<C>
sdk.buildDynamicAuction()      // → DynamicAuctionBuilder<C>
sdk.buildMulticurveAuction()   // → MulticurveBuilder<C>
sdk.buildOpeningAuction()      // → OpeningAuctionBuilder<C>
```

### Token / governance helpers

| Method | Returns |
|--------|---------|
| `getAirlockOwner()` | `Promise<Address>` |
| `getAirlockBeneficiary(shares?)` | `Promise<BeneficiaryData>` — defaults to 5% (0.05e18 WAD) |
| `getPoolInfo(poolAddress)` | `Promise<PoolInfo>` |
| `getHookInfo(hookAddress)` | `Promise<HookInfo>` |

### Entity getters

These return entity instances bound to a specific on-chain address.

| Method | Returns |
|--------|---------|
| `getStaticAuction(poolAddress)` | `Promise<StaticAuction>` |
| `getDynamicAuction(hookAddress)` | `Promise<DynamicAuction>` |
| `getMulticurvePool(tokenAddress)` | `Promise<MulticurvePool>` |
| `getRehypeDopplerHook(hookAddress)` | `Promise<RehypeDopplerHook>` |
| `getOpeningAuction(hookAddress)` | `Promise<OpeningAuction>` |
| `getOpeningAuctionLifecycle(initializerAddress?)` | `Promise<OpeningAuctionLifecycle>` — falls back to chain config; throws if unconfigured |
| `getOpeningAuctionPositionManager(positionManagerAddress?)` | `Promise<OpeningAuctionPositionManager>` — falls back to chain config; throws if unconfigured |
| `getOpeningAuctionBidManager({ openingAuctionHookAddress, openingAuctionPoolKey, positionManagerAddress? })` | `Promise<OpeningAuctionBidManager>` |
| `getDerc20(tokenAddress)` | `Derc20` |

***

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
* `withRehypeDopplerHook({ hookAddress, buybackDestination, customFee, assetBuybackPercentWad, numeraireBuybackPercentWad, beneficiaryPercentWad, lpPercentWad, graduationCalldata? })`
  * `hookAddress` — Deployed RehypeDopplerHook (must be whitelisted)
  * `buybackDestination` — Receives bought-back tokens
  * `customFee` — Swap fee in bps (3000 = 0.3%)
  * `assetBuybackPercentWad` — % for asset buyback (WAD, 1e18 = 100%)
  * `numeraireBuybackPercentWad` — % for numeraire buyback (WAD)
  * `beneficiaryPercentWad` — % for beneficiaries (WAD)
  * `lpPercentWad` — % for LPs (WAD)
  * `graduationCalldata` — Optional calldata executed on graduation
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

## OpeningAuctionBuilder

An auction used to place single sided LP positions in advance of a Doppler Dynamic auction. It can be used to mitigate sniping or more effectively set the clearing price prior to other price discovery auctions. Obtain via `sdk.buildOpeningAuction()`.

Methods (chainable):

* `tokenConfig(params)` — standard or doppler404 token
  * Standard: `{ name, symbol, tokenURI, yearlyMintRate? }`
  * Doppler404: `{ type: 'doppler404', name, symbol, baseURI, unit? }`
* `saleConfig({ initialSupply, numTokensToSell, numeraire })` — `numTokensToSell ≤ initialSupply`
* `openingAuctionConfig(params: OpeningAuctionConfig)`:
  * `auctionDuration` — opening auction duration in seconds (positive integer)
  * `tickSpacing` — opening auction pool tick spacing
  * `fee` — opening auction pool fee (0–`V4_MAX_FEE`)
  * `minAcceptableTickToken0` / `minAcceptableTickToken1` — int24 price floor/ceiling
  * `minLiquidity` — minimum liquidity the auction must attract (bigint, positive)
  * `shareToAuctionBps` — share of `numTokensToSell` allocated to the opening pool (1–10000)
  * `incentiveShareBps` — bidder incentive share in bps (0–10000)
* `dopplerConfig(params: OpeningAuctionDopplerConfig)`:
  * `minProceeds`, `maxProceeds` — soft and hard raise targets (bigint)
  * `startTick`, `endTick` — Doppler price range (direction depends on numeraire token ordering)
  * `duration?` — Doppler auction duration in seconds
  * `epochLength?` — Doppler epoch length in seconds; must divide `duration` evenly
  * `gamma?` — tick step per epoch; computed optimally when omitted
  * `numPdSlugs?` — number of price-discovery slugs
  * `fee?` — Doppler pool fee
  * `tickSpacing?` — Doppler pool tick spacing; must divide `openingAuction.tickSpacing` evenly
* `withVesting({ duration?, cliffDuration?, recipients?, amounts? })`
* `withGovernance(params: GovernanceOption<C>)` — `{ type: 'default' | 'noOp' | 'launchpad' | 'custom' }`
* `withMigration(migration: MigrationConfig)` — required; e.g. `{ type: 'uniswapV2' }`
* `withUserAddress(address)` — required
* `withIntegrator(address?)` — optional integrator fee recipient
* `withGasLimit(gas?)` — optional gas override (bigint)
* `withTime({ startTimeOffset?, startingTime?, blockTimestamp? })` — mutually exclusive: use `startTimeOffset` (seconds from now) or `startingTime` (absolute unix timestamp or Date)
* Module overrides (advanced): `withOpeningAuctionInitializer(address)`, `withOpeningAuctionPositionManager(address)`, `withAirlock(address)`, `withTokenFactory(address)`, `withDopplerDeployer(address)`, `withGovernanceFactory(address)`, `withV2Migrator(address)`, `withV4Migrator(address)`, `withNoOpMigrator(address)`
* `build()` → `CreateOpeningAuctionParams` — validates all constraints; throws descriptive errors on invalid config

Factory method:

```ts
const { hookAddress, tokenAddress, poolId } = await sdk.factory.createOpeningAuction(params)
```

***

## Factory methods

```ts
const { poolAddress, tokenAddress } = await sdk.factory.createStaticAuction(params)
const { hookAddress, tokenAddress, poolId } = await sdk.factory.createDynamicAuction(params)
const { poolId, tokenAddress } = await sdk.factory.createMulticurve(params)
const { hookAddress, tokenAddress, poolId } = await sdk.factory.createOpeningAuction(params)
```

***

## Solana SDK

### Initializer

The `initializer` namespace handles Doppler launches on Solana.

**`createInitializeLaunchInstruction(accounts, args)`**

Builds the on-chain instruction for a new Doppler launch.

Accounts (key fields):

| Account | Description |
|---------|-------------|
| `config` | Global initializer config PDA |
| `launch` | New launch PDA |
| `launchAuthority` | Launch authority PDA |
| `baseMint` | New base token mint keypair (signer) |
| `quoteMint` | Numeraire mint (e.g. WSOL) |
| `baseVault` / `quoteVault` | Token vault keypairs (signers) |
| `payer` / `authority` | Fee payer and launch authority (signers) |
| `migratorProgram` | Migrator program ID (e.g. `CPMM_MIGRATOR_PROGRAM_ID`) |
| `metadataAccount` | Token metadata account |
| `addressLookupTable` | ALT address for transaction compression (use `DOPPLER_DEVNET_ALT` on devnet) |

Args (key fields):

| Arg | Type | Description |
|-----|------|-------------|
| `namespace` | `Address` | Namespace for PDA uniqueness (typically payer address) |
| `launchId` | `Uint8Array` | 8-byte launch ID (from `launchIdFromU64`) |
| `baseDecimals` | `number` | Decimals of the base token |
| `baseTotalSupply` | `bigint` | Total base token supply (with decimals) |
| `baseForDistribution` | `bigint` | Tokens reserved for creator at graduation |
| `baseForLiquidity` | `bigint` | Tokens reserved for post-graduation liquidity |
| `curveVirtualBase` | `bigint` | XYK virtual base reserves (from `marketCapToCurveParams`) |
| `curveVirtualQuote` | `bigint` | XYK virtual quote reserves (from `marketCapToCurveParams`) |
| `curveFeeBps` | `number` | Swap fee during bonding curve phase (e.g. 100 = 1%) |
| `curveKind` | `number` | Curve type — use `CURVE_KIND_XYK` |
| `curveParams` | `Uint8Array` | Curve encoding — use `new Uint8Array([CURVE_PARAMS_FORMAT_XYK_V0])` |
| `migratorProgram` | `Address` | Migrator program that handles graduation |
| `migratorInitCalldata` | `Uint8Array` | Encoded graduation params (from `cpmmMigrator.encodeRegisterLaunchCalldata`) |
| `migratorMigrateCalldata` | `Uint8Array` | Encoded migration args (from `cpmmMigrator.encodeMigrateCalldata`) |
| `metadataName` / `metadataSymbol` / `metadataUri` | `string` | On-chain token metadata |

After building the instruction, append the `cpmmMigratorState` PDA as a writable remaining account so the register\_launch CPI can write graduation parameters:

```ts
const [cpmmMigratorState] = await cpmmMigrator.getCpmmMigratorStateAddress(launch)

const baseLaunchIx = createInitializeLaunchInstruction(accounts, args)

const launchIx = {
  ...baseLaunchIx,
  accounts: [
    ...(baseLaunchIx.accounts ?? []),
    { address: cpmmMigratorState, role: ACCOUNT_ROLE_WRITABLE },
  ],
}
```

***

### CPMM Migrator

The `cpmmMigrator` namespace encodes the calldata forwarded to the CPMM migrator at graduation.

**`encodeRegisterLaunchCalldata(args)`** → `Uint8Array` (`migratorInitCalldata`)

Encodes the `migratorInitCalldata` passed to `createInitializeLaunchInstruction`. Called once at launch creation to register graduation parameters.

```ts
const migratorInitCalldata = cpmmMigrator.encodeRegisterLaunchCalldata({
  cpmmConfig:              CPMM_CONFIG,   // Address of the CPMM AmmConfig
  initialSwapFeeBps:       30,            // Swap fee on graduated CPMM pool (0.3%)
  initialFeeSplitBps:      5000,          // % of fees distributed to LPs (50%)
  recipients: [
    { wallet: creatorAddress, amount: BASE_FOR_DISTRIBUTION },
  ],
  minRaiseQuote:           50_000_000_000n, // Graduation threshold in lamports (50 SOL)
  minMigrationPriceQ64Opt: null,            // Optional minimum graduation price floor
})
```

**`encodeMigrateCalldata(args)`** → `Uint8Array` (`migratorMigrateCalldata`)

Encodes the `migratorMigrateCalldata` passed to `createInitializeLaunchInstruction`. Forwarded at graduation time.

```ts
const migratorMigrateCalldata = cpmmMigrator.encodeMigrateCalldata({
  baseForDistribution: BASE_FOR_DISTRIBUTION,
  baseForLiquidity:    BASE_FOR_LIQUIDITY,
})
```

***

### CPMM

After graduation the bonding curve becomes a CPMM pool. These are the core helpers for swaps.

**`createSwapInstruction(accounts)` / `createSwapExactInInstruction(accounts, args)`**

`createSwapInstruction` is a convenience wrapper; `createSwapExactInInstruction` is the low-level form.

```ts
const ix = createSwapInstruction({
  config,
  pool:        poolAddress,
  authority:   pool.authority,
  vault0:      pool.vault0,
  vault1:      pool.vault1,
  token0Mint:  pool.token0Mint,
  token1Mint:  pool.token1Mint,
  userToken0:  userAta0,
  userToken1:  userAta1,
  user:        payer.address,
  amountIn:    1_000_000n,
  minAmountOut,
  direction:   0,
})
```

**Swap quote (off-chain, no RPC)**

```ts
const quote = getSwapQuote(pool, amountIn, direction)
// direction: 0 = token0→token1, 1 = token1→token0
// quote: { amountOut, feeTotal, feeDist, feeComp, priceImpact, executionPrice }
```

**Q64.64 fixed-point helpers**

```ts
numberToQ64(n: number): bigint   // multiply by 2^64
q64ToNumber(q: bigint): number   // divide by 2^64
```

**Pool fetching**

```ts
const [config]      = await getConfigAddress()
const [poolAddress] = await getPoolAddress(config, token0Mint, token1Mint)

// By address
const pool = await fetchPool(rpc, poolAddress)         // Pool | null

// By token pair (order-independent)
const result = await getPoolByMints(rpc, mint0, mint1) // { address, account: Pool } | null
```

