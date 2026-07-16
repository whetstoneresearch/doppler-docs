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

Import Solana helpers from the package subpath:

```ts
import {
  createLaunch,
  cpmm,
  cpmmHook,
  cpmmMigrator,
  initializer,
  deriveSolanaCpmmDeployment,
  DOPPLER_SOLANA_DEVNET_PROGRAM_ADDRESSES,
} from '@whetstone-research/doppler-sdk/solana'
```

### Deployment helpers

Use `deriveSolanaCpmmDeployment` to resolve the program IDs and config accounts used by the Solana helpers:

```ts
const deployment = await deriveSolanaCpmmDeployment(
  DOPPLER_SOLANA_DEVNET_PROGRAM_ADDRESSES,
)
```

The SDK exposes the dynamic fee hook as `cpmmHook` and its deployment address as `cpmmHookProgram`. The deployment object also includes the core protocol program IDs and the derived CPMM and initializer config accounts. For custom deployments, provide `cpmmHookProgram` for new launches.

### Initializer

The `initializer` namespace handles Doppler launches on Solana.

**`createLaunch(input)`**

Builds a complete `initialize_launch` instruction for a new Doppler launch. It derives launch PDAs, builds CPMM migration payloads by default, installs the dynamic fee hook, resolves hook flags, encodes hook payloads, and commits the relevant remaining-account hashes. Hook behavior is selected through the optional feature inputs below; there is no separate hook selector.

Key hook inputs:

| Field | Type | Description |
|-------|------|-------------|
| `dynamicFee` | `DynamicFeeScheduleArgs \| null` | Optional per-launch fee schedule stored in the hook payload. |
| `cosigner` | `AddressOrSigner` | Optionally enables cosigner gating through the dynamic fee hook. |
| `cosignGateExpiresAt` | `bigint \| number \| null` | Optional Unix timestamp after which the cosigner signature is no longer required. Requires `cosigner`. |

Hook features compose independently:

| Features | New-launch inputs |
|----------|-------------------|
| Neither | Omit `dynamicFee` and `cosigner` |
| Cosigning | Set `cosigner` |
| Dynamic fees | Set `dynamicFee` |
| Both | Set `dynamicFee` and `cosigner` |

Migration configuration is independent of hook features. CPMM migration can be enabled with any of the four feature combinations above.

Dynamic fees and cosigning can be combined in one hook:

```ts
const { instruction, addresses } = await createLaunch({
  deployment,
  launchAccounts: {
    baseMint,
    quoteMint,
    baseVault,
    quoteVault,
  },
  payer,
  authority: payer,
  supply: {
    baseDecimals: 6,
    baseTotalSupply: 1_000_000_000n * 10n ** 6n,
    baseForDistribution: 0n,
    baseForLiquidity: 0n,
  },
  curve: {
    curveVirtualBase: 1_000_000_000n * 10n ** 6n,
    curveVirtualQuote: 10n * 1_000_000_000n,
    swapFeeBps: 200,
  },
  dynamicFee: {
    startingTime: 0n,
    startFeeBps: 8_000,
    endFeeBps: 200,
    durationSeconds: 10n * 60n,
  },
  cosigner,
  cosignGateExpiresAt,
  migration: {
    minRaiseQuote: 50n * 1_000_000_000n,
  },
  metadata: null,
  feeBeneficiaries: [{ wallet: payer.address, shareBps: 10_000 }],
})
```

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
| `launchFeeState` | Launch fee state PDA |
| `payer` / `authority` | Fee payer and launch authority (signers) |
| `hookProgram` | Dynamic fee hook program ID for new launches |
| `migratorProgram` | Migrator program ID (e.g. `CPMM_MIGRATOR_PROGRAM_ID`) |
| `cpmmConfig` | CPMM config address when using the CPMM migrator |
| `baseTokenProgram` / `quoteTokenProgram` | Token program IDs for each mint |
| `metadataAccount` | Token metadata account, required when `metadataName` is non-empty |
| `hookCreateRemainingAccounts` | Readonly accounts forwarded to create hooks when `HF_BEFORE_CREATE` or `HF_AFTER_CREATE` is enabled |

Args (key fields):

| Arg | Type | Description |
|-----|------|-------------|
| `namespace` | `Address` | Namespace for PDA uniqueness (typically payer address) |
| `launchId` | `Uint8Array` | 32-byte launch ID, typically from `initializer.createLaunchId()` |
| `baseDecimals` | `number` | Decimals of the base token |
| `baseTotalSupply` | `bigint` | Total base token supply (with decimals) |
| `baseForDistribution` | `bigint` | Tokens reserved for creator at graduation |
| `baseForLiquidity` | `bigint` | Tokens reserved for post-graduation liquidity |
| `curveVirtualBase` | `bigint` | XYK virtual base reserves (from `marketCapToCurveParams`) |
| `curveVirtualQuote` | `bigint` | XYK virtual quote reserves (from `marketCapToCurveParams`) |
| `swapFeeBps` | `number` | Swap fee during bonding curve phase (e.g. 100 = 1%) |
| `curveKind` | `number` | Curve type — use `CURVE_KIND_XYK` |
| `curveParams` | `Uint8Array` | Curve encoding — use `new Uint8Array([CURVE_PARAMS_FORMAT_XYK_V0])` |
| `allowBuy` / `allowSell` | `boolean` | Enables curve buys and sells |
| `hookFlags` | `number` | Hook flags, e.g. `HF_BEFORE_SWAP` |
| `hookPayload` | `Uint8Array` | Hook payload forwarded to the hook program |
| `hookCreateRemainingAccountsLen` | `number` | Number of create-hook remaining accounts at the start of the routed remaining-account list |
| `hookCreateRemainingAccountsHash` | `Uint8Array` | Hash of create-hook remaining accounts |
| `hookRemainingAccountsHash` | `Uint8Array` | Hash of swap hook remaining accounts |
| `migratorInitPayload` | `Uint8Array` | Encoded graduation params (from `cpmmMigrator.encodeRegisterLaunchPayload`) |
| `migratorMigratePayload` | `Uint8Array` | Encoded migration args (from `cpmmMigrator.encodeMigratePayload`) |
| `migratorInitRemainingAccountsHash` | `Uint8Array` | Hash of migrator init remaining accounts |
| `migratorRemainingAccountsHash` | `Uint8Array` | Hash of migration remaining accounts |
| `feeBeneficiaries` | `Array<{ wallet, shareBps }>` | Curve fee beneficiaries |
| `metadataName` / `metadataSymbol` / `metadataUri` | `string` | On-chain token metadata |

#### Solana dynamic fee hook payloads

The dynamic fee hook is the supported hook for new launches. It can act as a pass-through hook, set a per-launch fee schedule, require a cosigner, or do both. When a schedule is present, launches should enable:

```ts
initializer.HF_BEFORE_CREATE | initializer.HF_BEFORE_SWAP
```

Add `initializer.HF_FORWARD_READONLY_SIGNERS` when the same dynamic fee hook launch also requires a cosigner. The dynamic fee hook stores the schedule in the launch hook payload; it does not require a schedule account.

The 32-byte schedule payload layout is:

| Bytes | Value |
|-------|-------|
| `0..8` | Magic bytes: `DFEEV1__` |
| `8` | Version, currently `1` |
| `9..16` | Reserved padding |
| `16..24` | Little-endian `i64` Unix timestamp `startingTime` |
| `24..26` | Little-endian `u16` `startFeeBps` |
| `26..28` | Little-endian `u16` `endFeeBps` |
| `28..32` | Little-endian `u32` `durationSeconds` |

`startingTime: 0` means "start when the launch is created." During `BEFORE_CREATE`, the hook normalizes `0` or a past timestamp to the current Solana clock timestamp and the initializer stores that normalized payload on the launch account. Future timestamps are preserved.

The hook returns the greater of the dynamic schedule fee and the launch's static `swapFeeBps`, so the schedule cannot reduce the fee below the launch's configured static fee.

The presence of the cosigner config account enables gating. The gate payload only determines whether and when that gate expires, so an indefinite gate does not need an expiry payload.

Payloads are composed as:

```text
no schedule or cosigner:              []
dynamic fee only:                    [32-byte schedule]
dynamic fee + indefinite cosigner:   [32-byte schedule]
dynamic fee + expiring cosigner:     [32-byte schedule][42-byte expiry payload]
indefinite cosigner only in this hook: []
expiring cosigner only in this hook: [42-byte expiry payload]
```

For low-level builders, dynamic-fee-only launches commit swap hook remaining accounts as:

```text
[namespace]
```

Dynamic fee launches that also require cosigning commit:

```text
[namespace, cosigner_config, cosigner]
```

If `namespace` equals `cosigner_config`, include that address only once:

```text
[cosigner_config, cosigner]
```

The create-hook remaining account list is empty:

```text
[]
```

When using `createInitializeLaunchInstruction` directly with `HF_BEFORE_CREATE`, set `hookCreateRemainingAccountsHash` to `initializer.computeRemainingAccountsHash([])`. The all-zero hash means "no create hook commitment" and is rejected when create hooks are enabled.

The SDK exposes helpers for encoding and inspection:

```ts
const payload = cpmmHook.encodeCpmmHookPayload({
  schedule: {
    startingTime: 0n,
    startFeeBps: 8_000,
    endFeeBps: 200,
    durationSeconds: 10n * 60n,
  },
})

cpmmHook.isDynamicFeeSchedulePayload(payload)
```

When `cpmmConfig` is provided, the SDK appends the CPMM migrator init remaining accounts automatically. Use the migrator helper to build the migration account list and committed hash:

```ts
const deployment = await deriveSolanaCpmmDeployment(
  DOPPLER_SOLANA_DEVNET_PROGRAM_ADDRESSES,
)

const migrationAccounts =
  await cpmmMigrator.buildCpmmMigrationRemainingAccounts({
    launch,
    baseMint,
    quoteMint,
    launchAuthority,
    adminBaseAta,
    adminQuoteAta,
    recipientAtas: [],
    cpmmProgram: deployment.cpmmProgram,
    cpmmMigratorProgram: deployment.cpmmMigratorProgram,
  })

const migratorInitRemainingAccountsHash =
  initializer.computeRemainingAccountsHash([
    migrationAccounts.cpmmMigrationState,
    migrationAccounts.cpmmConfig,
  ])
```

***

### CPMM Migrator

The `cpmmMigrator` namespace encodes the calldata forwarded to the CPMM migrator at graduation.

**`buildCpmmMigrationRemainingAccounts(args)`**

Derives the CPMM pool graph, account metas, and `migratorRemainingAccountsHash` used by `initialize_launch` and `migrate_launch`.

```ts
const migrationAccounts =
  await cpmmMigrator.buildCpmmMigrationRemainingAccounts({
    launch,
    baseMint,
    quoteMint,
    launchAuthority,
    adminBaseAta,
    adminQuoteAta,
    recipientAtas,
    cpmmProgram: deployment.cpmmProgram,
    cpmmMigratorProgram: deployment.cpmmMigratorProgram,
  })
```

**`encodeRegisterLaunchPayload(args)`** → `Uint8Array` (`migratorInitPayload`)

Encodes the `migratorInitPayload` passed to `createInitializeLaunchInstruction`. Called once at launch creation to register graduation parameters.

```ts
const migratorInitPayload = cpmmMigrator.encodeRegisterLaunchPayload({
  cpmmConfig:              migrationAccounts.cpmmConfig,
  initialSwapFeeBps:       30,            // Swap fee on graduated CPMM pool (0.3%)
  initialFeeSplitBps:      5000,          // % of fees distributed to LPs (50%)
  recipients: [
    { wallet: creatorAddress, amount: BASE_FOR_DISTRIBUTION },
  ],
  minRaiseQuote:           50_000_000_000n, // Graduation threshold in lamports (50 SOL)
  minMigrationPriceQ64Opt: null,            // Optional minimum graduation price floor
  migratedPoolHookConfig:  null,
})
```

**`encodeMigratePayload(args)`** → `Uint8Array` (`migratorMigratePayload`)

Encodes the `migratorMigratePayload` passed to `createInitializeLaunchInstruction`. Forwarded at graduation time.

```ts
const migratorMigratePayload = cpmmMigrator.encodeMigratePayload({
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
const ix = cpmm.createSwapInstruction({
  config,
  pool:        poolAddress,
  authority:   pool.authority,
  vault0:      pool.vault0,
  vault1:      pool.vault1,
  token0Mint:  pool.token0Mint,
  token1Mint:  pool.token1Mint,
  userToken0:  userAta0,
  userToken1:  userAta1,
  user:        payer,
  amountIn:    1_000_000n,
  minAmountOut,
  tradeDirection: 0,
  programId: deployment.cpmmProgram,
})
```

**Swap quote (off-chain, no RPC)**

```ts
const quote = cpmm.getSwapQuote(pool, amountIn, tradeDirection)
// tradeDirection: 0 = token0→token1, 1 = token1→token0
// quote: { amountOut, feeTotal, feeDist, feeComp, priceImpact, executionPrice }
```

**Q64.64 fixed-point helpers**

```ts
const q64 = cpmm.numberToQ64(1.5)
const n = cpmm.q64ToNumber(q64)
```

**Pool fetching**

```ts
const deployment = await deriveSolanaCpmmDeployment(
  DOPPLER_SOLANA_DEVNET_PROGRAM_ADDRESSES,
)
const [poolAddress] = await cpmm.getPoolAddress(
  token0Mint,
  token1Mint,
  deployment.cpmmProgram,
)

// By address
const pool = await cpmm.fetchPool(rpc, poolAddress, {
  programId: deployment.cpmmProgram,
})

// By token pair (order-independent)
const result = await cpmm.getPoolByMints(rpc, mint0, mint1, {
  programId: deployment.cpmmProgram,
})
```
