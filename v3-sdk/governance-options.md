---
icon: building-columns
---

# Governance Options

This guide explains how to configure different governance options when creating tokens with the Doppler V3 SDK, including using the NoOpGovernanceFactory for gas-efficient deployments.

## Overview

The Doppler V3 SDK supports optional governance with the following models.

1. **"Standard" Governance** - Full on-chain governance with timelock using OpenZeppelin Governor
2. **"No-Op" Governance** - Minimal governance for gas savings (sets governance to `0xdead`)

## Using NoOpGovernanceFactory

The NoOpGovernanceFactory creates tokens without active governance, significantly reducing deployment costs and complexity. This is ideal for projects that don't require on-chain governance.

### Prerequisites

Ensure the NoOpGovernanceFactory is deployed on your target chain. Currently available on:

* **Base Sepolia**: `0x916B8987E4aD325C10d58ED8Dc2036a6FF5EB228`

### Implementation

```typescript
import { ReadWriteFactory, CreateV3PoolParams } from 'doppler-v3-sdk';
import { DOPPLER_V3_ADDRESSES } from 'doppler-v3-sdk';

// Get addresses for your chain
const chainId = 84532; // Base Sepolia
const addresses = DOPPLER_V3_ADDRESSES[chainId];

// Check if NoOpGovernanceFactory is available
if (!addresses.noOpGovernanceFactory) {
  throw new Error('NoOpGovernanceFactory not deployed on this chain');
}

// Create parameters with NoOpGovernanceFactory
const createParams: CreateV3PoolParams = {
  integrator: '0x...', // Your integrator address
  userAddress: '0x...', // User creating the token
  numeraire: '0x...', // Base token (e.g., WETH)
  contracts: {
    tokenFactory: addresses.tokenFactory,
    // Use NoOpGovernanceFactory instead of standard governanceFactory
    governanceFactory: addresses.noOpGovernanceFactory,
    v3Initializer: addresses.v3Initializer,
    liquidityMigrator: addresses.liquidityMigrator,
  },
  tokenConfig: {
    name: 'My Token',
    symbol: 'MTK',
    tokenURI: 'https://example.com/metadata.json',
  },
  vestingConfig: 'default',
  // Optional: Configure V4 migration
  liquidityMigratorData: '0x...', // See V4 migrator docs
};

// Create the factory instance
const factory = new ReadWriteFactory(addresses.airlock, driftClient);

// Encode and create the token
const createData = await factory.encodeCreateData(createParams);
```

## Using Standard Governance

For tokens that require governance functionality:

```typescript
const createParams: CreateV3PoolParams = {
  // ... other parameters ...
  contracts: {
    tokenFactory: addresses.tokenFactory,
    // Use standard governanceFactory
    governanceFactory: addresses.governanceFactory,
    v3Initializer: addresses.v3Initializer,
    liquidityMigrator: addresses.liquidityMigrator,
  },
  // Optional: Customize governance parameters
  governanceConfig: {
    initialVotingDelay: 3600, // 1 hour
    initialVotingPeriod: 172800, // 48 hours
    initialProposalThreshold: 1000000n, // 1% of supply
  },
};
```

## Custom Governance Factory

To use a custom governance factory (must be whitelisted):

```typescript
const createParams: CreateV3PoolParams = {
  // ... other parameters ...
  contracts: {
    tokenFactory: addresses.tokenFactory,
    // Use your custom governance factory address
    governanceFactory: '0xYourCustomGovernanceFactory',
    v3Initializer: addresses.v3Initializer,
    liquidityMigrator: addresses.liquidityMigrator,
  },
};
```

## Fee Distribution

### Standard Governance (90/10 Split)

* 90% of liquidity → Timelock (controlled by governance)
* 10% of liquidity → StreamableFeesLocker (distributed to beneficiaries)

### No-Op Governance (100% Locked)

For permanent liquidity provision, set the recipient to `DEAD_ADDRESS`:

```typescript
import { DEAD_ADDRESS } from "doppler-v3-sdk";

// In no-op governance, all liquidity goes to the locker
const recipient = DEAD_ADDRESS; // 0x000...dEaD
```



## See Also

* [V4 Migrator Guide](custom-fees.md) - Configure V4 migration with beneficiaries
* [Token Launch Examples](../doppler-v3-sdk-reference/token-launch-examples.md) - Complete examples
* [Contract Addresses](../doppler-v3-sdk-reference/contract-addresses.md) - Deployment addresses by chain
