---
icon: building-columns
---

# Governance Options

This guide explains how to configure different governance options when creating tokens with the Doppler V4 SDK, including using the NoOpGovernanceFactory for gas-efficient deployments.

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

