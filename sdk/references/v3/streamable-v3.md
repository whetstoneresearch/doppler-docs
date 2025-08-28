---
icon: water
description: Streamable V3 (Lockable V3 Pools) Reference
---

# Streamable V3 Pools

## Overview

Streamable V3 pools use the `LockableUniswapV3Initializer` contract to create Uniswap V3 pools with fee streaming capabilities. This feature allows projects to distribute trading fees to multiple beneficiaries over time, similar to the fee streaming available in Doppler V4 pools.

## Key Concepts

### What Makes a Pool "Streamable"?

A streamable V3 pool is created when you specify beneficiaries during pool initialization. These pools have the following characteristics:

2. **Fee Distribution**: Trading fees are collected and distributed to beneficiaries based on their share percentages
3. **No Graduation**: Unlike standard V3 pools, streamable pools never "graduate" or migrate to another pool type
4. **Continuous Revenue**: Beneficiaries can claim their share of fees at any time by calling `collectFees()`

### When to Use Streamable V3

Consider using streamable V3 pools when:
- You want to distribute trading fees to multiple stakeholders
- You prefer a simpler, permanent pool structure without migration

## Technical Requirements

### Beneficiary Configuration

When creating a streamable V3 pool, you must configure beneficiaries with the following requirements:

```typescript
interface BeneficiaryData {
  beneficiary: Address;  // Recipient address
  shares: bigint;       // Share amount in WAD (1e18 = 100%)
}
```

**Requirements:**
1. **Total Shares**: Must sum to exactly 1e18 (100%)
2. **Protocol Fee**: The Airlock owner must receive exactly 5% (0.05e18) shares
3. **Ordering**: Beneficiaries must be sorted by address in ascending order
4. **Non-zero Shares**: Each beneficiary must have shares > 0

### Example Configuration

```typescript
import { parseEther } from 'viem';
import { ReadWriteFactory, DOPPLER_V3_ADDRESSES } from '@doppler-v3-sdk';

// Set up wallet (use your preferred method - WalletConnect, injected, etc.)
const walletClient = createWalletClient({
  // ... your wallet configuration
});

// Set up factory
const chainId = 8453; // Base mainnet
const factory = new ReadWriteFactory(
  DOPPLER_V3_ADDRESSES[chainId].airlock,
  DOPPLER_V3_ADDRESSES[chainId].bundler,
  drift // your configured drift instance
);

// Get the Airlock owner address
const airlockOwner = await factory.owner();

// Configure beneficiaries
const beneficiaries = [
  {
    beneficiary: airlockOwner,
    shares: parseEther("0.05")  // 5% protocol fee (required)
  },
  {
    beneficiary: "0xProjectTreasury...",
    shares: parseEther("0.50")  // 50% to project treasury
  },
  {
    beneficiary: "0xDevelopmentFund...",
    shares: parseEther("0.30")  // 30% to development
  },
  {
    beneficiary: "0xCommunityRewards...",
    shares: parseEther("0.15")  // 15% to community
  }
];

// Sort beneficiaries (required)
const sortedBeneficiaries = factory.sortBeneficiaries(beneficiaries);

// Create pool with streamable fees
const params = {
  // ... other parameters
  contracts: {
    tokenFactory: DOPPLER_V3_ADDRESSES[chainId].tokenFactory,
    governanceFactory: DOPPLER_V3_ADDRESSES[chainId].governanceFactory,
    v3Initializer: DOPPLER_V3_ADDRESSES[chainId].lockableV3Initializer, // Use lockable initializer
    liquidityMigrator: DOPPLER_V3_ADDRESSES[chainId].liquidityMigrator,
  },
  v3PoolConfig: {
    // ... standard config
    beneficiaries: sortedBeneficiaries // Add beneficiaries for streamable fees
  }
};

// Encode and create the pool
const { createParams } = factory.encode(params);
const txHash = await factory.create(createParams);
```

## Important Considerations

### No Migration Path

**Critical**: Pools created with beneficiaries will never migrate. This means:
- The pool remains on Uniswap V3 permanently
- Liquidity cannot be moved to V2 or V4
- The initial pool parameters cannot be changed
- Fee distribution continues indefinitely

### Fee Collection

Beneficiaries can collect their accumulated fees at any time by calling:
```typescript
const tx = await lockableV3Initializer.collectFees(poolAddress);
```

This will:
1. Collect all accumulated fees from the pool's liquidity positions
2. Distribute fees to each beneficiary according to their shares
3. Emit `Collect` events for each beneficiary

## Best Practices

1. **Plan Beneficiaries Carefully**: Once set, beneficiaries cannot be changed
3. **Test on Testnet**: Always test your beneficiary configuration on testnet first

## Migration from Standard V3 Pools

### Feature Comparison

| Feature | Standard V3 Pool | Streamable V3 Pool |
|---------|------------------|-------------------|
| **Initializer Contract** | `UniswapV3Initializer` | `LockableUniswapV3Initializer` |
| **Fee Distribution** | Accumulated in pool until migration | Continuously claimable by beneficiaries |
| **Pool Migration** | Graduates to V2 or V4 after conditions met | Never migrates (permanent V3) |
| **Beneficiaries** | Not supported | Multiple beneficiaries with custom shares |
| **Protocol Fee** | Collected on migration | 5% streamed to protocol continuously |
| **Liquidity Management** | Full range + migration planning | Full range permanently |
| **Use Case** | Standard token launches with future migration | Projects needing continuous fee distribution |

### When to Use Each Approach

**Use Standard V3 Pools When:**
- You want the flexibility to migrate to Uniswap V2 or V4 later
- You prefer consolidated liquidity in a single graduated pool
- You don't need immediate fee distribution
- You want to follow the traditional Doppler launch pattern

**Use Streamable V3 Pools When:**
- You need continuous revenue distribution to multiple parties
- You want a "set and forget" pool structure
- You have stakeholders who need regular fee access
- You prefer simplicity over migration flexibility

### Key Implementation Differences

The main differences when implementing streamable V3 pools:

1. **Initializer Address**: Use `lockableV3Initializer` instead of `v3Initializer`
2. **Beneficiaries**: Must be configured with shares summing to exactly 100%
3. **Liquidity Migrator**: Set to `zeroAddress` (migration is disabled)
4. **Unlock Time**: Ignored by the contract (pools never unlock)

### Example: Converting Configuration

**Standard V3 Configuration:**
```typescript
const standardConfig = {
  contracts: {
    v3Initializer: addresses.v3Initializer,
    liquidityMigrator: addresses.v2Migrator, // Or v4Migrator
  },
  v3PoolConfig: {
    feeTier: 10000,
    initialEthLiquidity: parseEther("2"),
    unlockTime: futureTimestamp,
    shouldLaunch: true
  }
};
```

**Streamable V3 Configuration:**
```typescript
const streamableConfig = {
  contracts: {
    v3Initializer: addresses.lockableV3Initializer, // Changed
    liquidityMigrator: zeroAddress, // Changed - no migration
  },
  v3PoolConfig: {
    feeTier: 10000,
    initialEthLiquidity: parseEther("2"),
    shouldLaunch: true,
    beneficiaries: sortedBeneficiaries // Added - enables fee streaming
    // unlockTime removed (ignored by contract)
  }
};
```

### Migration Checklist

If you're moving from standard V3 to streamable V3 for new deployments:

- [ ] Update initializer address to `lockableV3Initializer`
- [ ] Define beneficiaries with shares summing to exactly 100%
- [ ] Include 5% protocol fee to Airlock owner
- [ ] Sort beneficiaries using `factory.sortBeneficiaries()`
- [ ] Set liquidityMigrator to `zeroAddress`
- [ ] Remove unlock time planning (parameter is ignored)
- [ ] Update fee collection processes to use `collectFees()`
- [ ] Communicate permanent pool structure to stakeholders

## FAQ

**Q: Can I change beneficiaries after pool creation?**
A: No, beneficiaries are immutable once the pool is created.

**Q: Can I create a streamable pool without the 5% protocol fee?**
A: No, the contract enforces that the Airlock owner receives exactly 5% of fees.

**Q: Can I convert an existing standard V3 pool to streamable?**
A: No, pool types are determined at creation. You would need to create a new token.

**Q: What happens to tokens already launched with standard V3?**
A: They continue to work as designed and will migrate according to their configuration.

**Q: Can I use streamable V3 with V4 migration for fee streaming?**
A: No, streamable V3 pools never migrate. If you want V4 with fee streaming, use standard V3 + V4 migrator.
