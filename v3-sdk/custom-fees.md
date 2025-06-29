---
description: Arbitrary migration to Uniswap v4 pools with customizable fees
icon: arrow-right-arrow-left
---

# Custom Fees

The V3 SDK includes support for creating Doppler V3 pools that can migrate their liquidity to Uniswap V4 with customizable fee streaming. This allows protocols to distribute trading fees to multiple beneficiaries over time, and importantly, customize the post-graduation fee amounts.&#x20;

## Overview

The "V4 migrator" enables:

* Migration from v3 or v4 Doppler pools to Uniswap V4 pools
* Fee streaming to multiple beneficiaries with custom shares
* Time-locked liquidity with configurable duration
* Support for both standard and no-op governance models

## Usage with V3 SDK

Doppler v3 tokens can specify their usage of the Uniswap v4 migrator at the time of creation.&#x20;

### Integration with CreateV3PoolParams

The V3 SDK's `CreateV3PoolParams` interface now includes an optional `liquidityMigratorData` field:

```typescript
interface CreateV3PoolParams {
  // ... other parameters ...
  liquidityMigratorData?: Hex; // Encoded V4 migration configuration
}
```

### Step 1: Configure Beneficiaries

```typescript
import { BeneficiaryData, WAD } from "doppler-v3-sdk";

// Shares must sum to exactly WAD (1e18)
const beneficiaries: BeneficiaryData[] = [
  { 
    beneficiary: "0x...", // Treasury address
    shares: WAD * 70n / 100n  // 70% of fees
  },
  { 
    beneficiary: "0x...", // Development fund
    shares: WAD * 20n / 100n  // 20% of fees
  },
  { 
    beneficiary: "0x...", // Community rewards
    shares: WAD * 10n / 100n  // 10% of fees
  }
];
```

### Step 2: Sort and Validate Beneficiaries

Beneficiaries must be sorted by address in ascending order:

```typescript
import { ReadWriteFactory } from "doppler-v3-sdk";

const factory = new ReadWriteFactory(airlockAddress, bundlerAddress);
const sortedBeneficiaries = factory.sortBeneficiaries(beneficiaries);
```

### Step 3: Configure V4 Migrator

```typescript
import { V4MigratorData } from "doppler-v3-sdk";

const migratorConfig: V4MigratorData = {
  fee: 3000,                        // 0.3% fee tier
  tickSpacing: 60,                  // Tick spacing for the V4 pool
  lockDuration: 365 * 24 * 60 * 60, // 1 year lock
  beneficiaries: sortedBeneficiaries
};
```

### Step 4: Encode Configuration

```typescript
const encodedMigratorData = factory.encodeV4MigratorData(migratorConfig);
```

### Step 5: Create Pool with V4 Migration

```typescript
const createParams = await factory.buildConfig({
  integrator: "0x...",
  userAddress: userAddress,
  numeraire: addresses.unichain.weth,
  contracts: {
    tokenFactory: addresses.unichain.tokenFactory,
    governanceFactory: addresses.unichain.governanceFactory,
    poolInitializer: addresses.unichain.v3Initializer,
    liquidityMigrator: addresses.unichain.liquidityMigrator, // V4 migrator
  },
  tokenConfig: {
    name: "My Token",
    symbol: "MTK",
    tokenURI: "https://example.com/token-metadata.json"
  },
  // Pass the encoded migrator data
  liquidityMigratorData: encodedMigratorData,
  // ... other parameters
});

await factory.create(createParams);
```

## Querying Migrator State

Use the `ReadMigrator` class to query migrator configuration:

```typescript
import { ReadMigrator } from "doppler-v3-sdk";

const migrator = new ReadMigrator(migratorAddress);

// Get V4 pool configuration
const poolKey = await migrator.getAssetData(token0, token1);
console.log(`Fee tier: ${poolKey.fee}`);
console.log(`Tick spacing: ${poolKey.tickSpacing}`);

// Get contract addresses
const locker = await migrator.locker();           // StreamableFeesLocker
const poolManager = await migrator.poolManager(); // V4 PoolManager
const airlock = await migrator.airlock();         // Airlock address
```



## Important Considerations

### Beneficiary Requirements

1. **Sorted Order**: Beneficiaries must be sorted by address (ascending)
2. **Positive Shares**: All shares must be greater than 0
3. **Sum to WAD**: Total shares must equal exactly 1e18 (100%)

### Fee Tiers and Tick Spacing

Common V4 configurations:

* 0.01% fee → 1 tick spacing
* 0.05% fee → 10 tick spacing
* 0.3% fee → 60 tick spacing
* 1% fee → 200 tick spacing

### Lock Duration

* Minimum: No minimum (can be 0 for immediate unlocking)
* Maximum: No maximum (can be set to centuries for permanent locks)
* Typical: 1-4 years for protocol-owned liquidity

## Helper Functions

### sortBeneficiaries

```typescript
const sorted = factory.sortBeneficiaries(beneficiaries);
```

Sorts beneficiaries by address in ascending order (required by contract).

### encodeV4MigratorData

```typescript
const encoded = factory.encodeV4MigratorData(migratorConfig);
```

Encodes the V4 migrator configuration for use in pool creation. Validates:

* Beneficiaries are sorted
* All shares are positive
* Total shares equal WAD

## Migration Flow

1. **Pool Creation**: Doppler V3 pool is created with initial liquidity
2. **Trading Phase**: Users trade in the V3 pool during the sale period
3. **Migration Trigger**: When conditions are met (time/volume), migration begins
4. **Liquidity Exit**: All liquidity is removed from the V3 pool
5. **V4 Pool Creation**: New V4 pool is created with full-range liquidity
6. **Fee Streaming**: Trading fees accumulate and stream to beneficiaries

## Example: Multi-Beneficiary Setup

```typescript
// Example: Protocol with multiple stakeholders
const beneficiaries: BeneficiaryData[] = [
  { 
    beneficiary: "0x123...", // DAO Treasury
    shares: WAD * 40n / 100n // 40%
  },
  { 
    beneficiary: "0x456...", // Core Team
    shares: WAD * 25n / 100n // 25%
  },
  { 
    beneficiary: "0x789...", // Ecosystem Fund
    shares: WAD * 20n / 100n // 20%
  },
  { 
    beneficiary: "0xabc...", // Bug Bounty Reserve
    shares: WAD * 10n / 100n // 10%
  },
  { 
    beneficiary: "0xdef...", // Community Incentives
    shares: WAD * 5n / 100n  // 5%
  }
];

// Create configuration with 2-year lock
const config: V4MigratorData = {
  fee: 10000,              // 1% fee for exotic pair
  tickSpacing: 200,        // Corresponding tick spacing
  lockDuration: 2 * 365 * 24 * 60 * 60, // 2 years
  beneficiaries: factory.sortBeneficiaries(beneficiaries)
};
```

## See Also

* [StreamableFeesLocker Reference](../v4-sdk/custom-fees.md)
* [Factory Reference](factory.md)
* [V4 SDK Documentation](../v4-sdk/factory.md)
