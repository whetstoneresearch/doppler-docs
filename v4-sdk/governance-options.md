---
icon: building-columns
---

# Governance Options

Similar to the [Doppler v3 SDK](../v3-sdk/governance-options.md) the Doppler v4 SDK support optional governance.&#x20;

## Overview

The Doppler V4 SDK supports optional governance with the following models.

1. **"Standard" Governance** - Full on-chain governance with timelock using OpenZeppelin Governor
2. **"No-Op" Governance** - Minimal governance for gas savings (sets governance to `0xdead`)

## Using NoOpGovernanceFactory

The NoOpGovernanceFactory creates tokens without active governance, significantly reducing deployment costs and complexity. This is ideal for projects that don't require on-chain governance.



### Examples

### 1. Standard Governance Configuration

```typescript
const config = await factory.buildConfig({
  // ... other parameters
  liquidityMigratorData,
  integrator: '0x...integrator',
}, addresses);

// Create the pool
const txHash = await factory.create(config.createParams);
```

### 2. No-Op Governance Configuration

For no-op governance (permanent liquidity lock with perpetual fee streaming):

```typescript
// Configure for no-op governance
const config = await factory.buildConfig({
  // ... other parameters
  liquidityMigratorData,
  integrator: '0x...integrator',
}, addresses, {
  useGovernance: false // This uses the noOpGovernanceFactory
});

// The migration will automatically set recipient to DEAD_ADDRESS (0xdead)
// This ensures the position is permanently locked
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
