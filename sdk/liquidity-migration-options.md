---
description: Learn more about how liquidity migration can be customized
icon: grate-droplet
---

# Liquidity migration options

## Overview

Doppler supports migrating liquidity from auction contracts to long-term AMM pools optimized for sustained trading. Migration triggers are configurable at creation, typically based on proceeds thresholds that indicate sufficient liquidity for the next growth phase. Currently supported migration targets include Uniswap v2, v3, and v4, with configurable fee structures. Support for additional AMMs and ecosystems is expected.

### Non-migration (&#x72;_&#x65;commended_)

In a variety of use cases, it may often make sense to _not_ migrate liquidity from the pool utilized to auction the token and bootstrap liquidity to another, longer term AMM pool. This can be done to maximize simplicity, for example, providing unified experiences on trading charts and terminals, or a variety of other reasons. To not migrate liquidity, use the `noOp` parameter in the migration config. Notably this is supported for launches using Doppler Multicurve or "Lockable" V3 static auctions.

## Migration Options Guide

The SDK encodes post‑auction liquidity migration via a discriminated union `MigrationConfig`:

```ts
export type MigrationConfig =
  | { type: 'noOp' }
  | { type: 'uniswapV2' }
  | { type: 'uniswapV3'; fee: number; tickSpacing: number }
  | {
      type: 'uniswapV4'
      fee: number
      tickSpacing: number
      streamableFees: {
        lockDuration: number // seconds
        beneficiaries: { beneficiary: Address; shares: bigint }[] // shares in WAD (1e18 = 100%)
      }
    }
```

Internally, the factory resolves the on‑chain migrator address for your chain and ABI‑encodes the specific data shape required by that migrator.

### Quick Decision Guide

* Want the simpliest solution and unified charts/UX?&#x20;
* Want simplest AMM and immediate trading? Use V2
* Want a concentrated liquidity range in the resulting pool? Use V3
* Want programmable fee streaming to beneficiaries and are on a V4‑ready chain? Use V4

### No Migration&#x20;

```ts
.withMigration({ type: 'noOp' })
```

* Supported on: UniswapV4MulticurveInitializer, LockableUniswapV3Initializer (Static auctions only)
* **Note:** `noOp` migration is NOT supported for Dynamic auctions - the SDK will throw an error if you try to use it

### V2 Migration

```ts
.withMigration({ type: 'uniswapV2' })
```

* Encoded data: empty (`0x`)
* Migrator address resolved per chain (see `src/addresses.ts`)

### V3 Migration

```ts
.withMigration({ type: 'uniswapV3', fee: 3000, tickSpacing: 60 })
```

* Encoded data: `(fee:uint24, tickSpacing:int24)`
* Ensure `tickSpacing` matches the selected `fee` tier on your chain

### V4 Migration (streamable fees)

```ts
.withMigration({
  type: 'uniswapV4',
  fee: 3000,
  tickSpacing: 60,
  streamableFees: {
    lockDuration: 365 * 24 * 60 * 60, // 1 year
    beneficiaries: [
      { beneficiary: '0x...', shares: parseEther('0.6') },  // 60%
      { beneficiary: '0x...', shares: parseEther('0.4') },  // 40%
    ],
  },
})
```

* Encoded data:
  * `(fee:uint24, tickSpacing:int24, lockDuration:uint32, beneficiaries: (address, shares[WAD])[])`
  * The SDK sorts beneficiaries by address (ascending) as required by the contract
* Validation:
  * At least one beneficiary
  * Shares must sum to exactly 1e18 (100%)
  * Contract enforces: airlock owner must receive at least 5% of streamed fees (add as a beneficiary if applicable)
* Chain support:
  * Ensure `streamableFeesLocker` and `v4Migrator` are deployed on your target chain (see `src/addresses.ts`)

### Governance Selection

* No‑op governance: Call `withGovernance({ type: 'noOp' })`. The SDK throws if `noOpGovernanceFactory` is not deployed on the chain.
* Standard governance: Call `withGovernance({ type: 'default' })` for standard defaults.
* Custom governance: Call `withGovernance({ type: 'custom', initialVotingDelay, initialVotingPeriod, initialProposalThreshold })`.

### Address Resolution

Migrator contracts are selected per chain via `getAddresses(chainId)` (see `src/addresses.ts`).

* `v2Migrator`, `v3Migrator`, `v4Migrator` must be present for the chosen type
* Some optional contracts (`noOpGovernanceFactory`, `streamableFeesLocker`) may be `0x0` on certain chains — avoid V4 migration with fee streaming where not supported. Using no‑op governance requires `noOpGovernanceFactory`.

### When to choose which

* No migration (noOp)
  * Simpliest possible solution & fastest time to market
  * Unified charts for pre and post liquidity bootstrapping
  * Reduced complexity, debugging, and testing requirements
  * Utilizing Multicurve or another supported initializer
* Uniswap V2
  * Basic constant‑product pool; broad ecosystem tooling
  * No price range configuration; least complexity
  * Good default if you do not require V3/V4‑specific features
* Uniswap V3
  * Concentrated liquidity with a fixed range
  * Requires `fee` tier and matching `tickSpacing`
  * Choose when you want to seed a V3 pool with explicit range after the sale
* Uniswap V4
  * Pools with hooks; supports fee streaming via `StreamableFeesLocker`
  * Requires `fee`, `tickSpacing`, and `streamableFees`
  * Choose when you want programmable fee distribution to beneficiaries, and V4 infra is available on your chain
