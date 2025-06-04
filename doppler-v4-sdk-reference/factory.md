---
icon: square-terminal
description: Factory Class Reference
---

# Factory

The Doppler V4 SDK provides two factory classes for interacting with the airlock contract:

- **`ReadFactory`** - Read-only operations for querying deployed pools and modules
- **`ReadWriteFactory`** - Extends ReadFactory with deployment and migration capabilities

## ReadFactory

The `ReadFactory` class provides read-only operations for the Doppler V4 airlock contract. It handles queries and data retrieval from deployed Doppler pools and their associated contracts.

### Constructor

```typescript
new ReadFactory(address: Address, drift: Drift<ReadAdapter>)
```

**Parameters:**

- `address` - The address of the airlock contract
- `drift` - Drift instance with read adapter (creates default if not provided)

### Methods

#### getModuleState

Retrieves the current state/type of a module in the Doppler system. Modules serve different roles and must be whitelisted before use.

```typescript
async getModuleState(module: Address): Promise<ModuleState>
```

**Parameters:**

- `module` - The address of the module to check

**Returns:** Promise resolving to the module's current state

**Module States:**

- `NotWhitelisted` (0) - Module is not approved for use
- `TokenFactory` (1) - Module can create tokens
- `GovernanceFactory` (2) - Module can create governance contracts
- `HookFactory` (3) - Module can create hooks
- `Migrator` (4) - Module can migrate liquidity

#### getAssetData

Retrieves comprehensive deployment data for a Doppler asset, including all contract addresses and configuration.

```typescript
async getAssetData(asset: Address): Promise<AssetData>
```

**Parameters:**

- `asset` - The address of the deployed asset token

**Returns:** Promise resolving to complete asset deployment data including:

- Numeraire (quote token) used for pricing
- Timelock and governance contracts
- Liquidity migrator for post-discovery trading
- Pool initializer and pool addresses
- Number of tokens being sold
- Integrator information

### ReadFactory Example

```typescript
import { ReadFactory, ModuleState } from "doppler-v4-sdk";

// Create factory instance
const factory = new ReadFactory(airlockAddress);

// Check module state
const state = await factory.getModuleState(moduleAddress);
if (state === ModuleState.TokenFactory) {
  console.log("Module is a valid token factory");
}

// Get asset information
const assetData = await factory.getAssetData(tokenAddress);
console.log("Asset details:", {
  numeraire: assetData.numeraire,
  governance: assetData.governance,
  pool: assetData.pool,
  migrationPool: assetData.migrationPool,
  totalSupply: assetData.totalSupply.toString(),
  tokensForSale: assetData.numTokensToSell.toString(),
});
```

## ReadWriteFactory

The `ReadWriteFactory` class extends `ReadFactory` with comprehensive deployment and migration capabilities for creating tokens with Doppler.

## Key Features

- **Pool Creation**: Deploy requisite doppler contracts, with tokens, hooks, and governance
- **Hook Mining**: Find optimal hook addresses with required flags
- **Asset Migration**: Move liquidity from price discovery to standard trading
- **Parameter Validation**: Automatic validation and optimization of deployment parameters
- **Gamma Calculation**: Compute optimal price movement parameters

## Constructor

```typescript
new ReadWriteFactory(address: Address, drift: Drift<ReadWriteAdapter>)
```

**Parameters:**

- `address` - The address of the airlock contract
- `drift` - A Drift instance with read-write adapter capabilities

## Core Methods

### buildConfig

Builds complete configuration for creating a new Doppler pool. This method validates parameters and tick ranges, optionally computes a valid gamma, mines hook addresses, and encodes factory data.

```typescript
buildConfig(
  params: DopplerPreDeploymentConfig,
  addresses: DopplerV4Addresses
): {
  createParams: CreateParams;
  hook: Hex;
  token: Hex;
}
```

**Parameters:**

- `params` - Pre-deployment configuration parameters
- `addresses` - Addresses of required Doppler V4 contracts

**Returns:** Object containing creation parameters, hook address, and token address

### create

Creates a new Doppler pool with token, hook, migrator, and governance. This is the main deployment method that sets up the complete ecosystem.

```typescript
async create(
  params: CreateParams,
  options?: TransactionOptions
): Promise<Hex>
```

**Parameters:**

- `params` - Complete creation parameters from `buildConfig()`
- `options` - Optional transaction options (gas, value, etc.)

**Returns:** Promise resolving to the transaction hash

### simulateCreate

Simulates a pool creation transaction without executing it. Useful for gas estimation, parameter validation, and testing configurations.

```typescript
async simulateCreate(
  params: CreateParams
): Promise<FunctionReturn<AirlockABI, 'create'>>
```

**Parameters:**

- `params` - Complete creation parameters from `buildConfig()`

**Returns:** Promise resolving to simulation results including gas estimates

### migrate

Migrates liquidity for an existing asset from the current pool to the migration pool. Triggers the migration process for assets that have completed price discovery.

```typescript
async migrate(
  asset: Address,
  options?: TransactionOptions
): Promise<Hex>
```

**Parameters:**

- `asset` - The address of the asset token to migrate
- `options` - Optional transaction options

**Returns:** Promise resolving to the transaction hash

## Example Usage

```typescript
import { ReadWriteFactory } from "doppler-v4-sdk";

// Create factory instance
const factory = new ReadWriteFactory(airlockAddress, drift);

// Build complete configuration
const { createParams, hook, token } = factory.buildConfig(
  {
    name: "MyToken",
    symbol: "MYT",
    totalSupply: parseEther("1000000000"), // 1b tokens
    numTokensToSell: parseEther("600000000"), // 600m tokens
    tokenURI: "some_ipfs_cid",
    blockTimestamp: Math.floor(Date.now() / 1000),
    startTimeOffset: 1,
    duration: 1 / 4, // 6 hour duration, must be divisible by epochLength
    epochLength: 200, // 200 seconds
    gamma: 800,
    tickRange: {
      startTick: 174_312,
      endTick: 186_840,
    },
    tickSpacing: 2,
    fee: 20_000, // 2%
    minProceeds: parseEther("2"),
    maxProceeds: parseEther("4"),
    yearlyMintRate: 0n,
    vestingDuration: BigInt(24 * 60 * 60 * 365),
    recipients: [someRecipientAddress],
    amounts: [parseEther("50000000")], // 5% of totalSupply
    numPdSlugs: 15,
    integrator: yourIntegratorAddress,
  },
  addresses
);

// Simulate creation to estimate gas
const simulation = await factory.simulateCreate(createParams);
console.log(`Estimated gas: ${simulation.request.gas}`);

// Create the pool
const txHash = await factory.create(createParams, {
  gasLimit: 5000000n,
});

// Later, migrate liquidity after price discovery ends
const migrationTx = await factory.migrate(token);
```

## Configuration Parameters

The `DopplerPreDeploymentConfig` includes:

- **Token Details**: `name`, `symbol`, `totalSupply`, `tokenURI`
- **Sale Parameters**: `numTokensToSell`, `duration`, `epochLength`
- **Price Discovery**: `tickRange`, `tickSpacing`, `gamma`, `fee`
- **Proceeds**: `minProceeds`, `maxProceeds`
- **Governance**: `yearlyMintRate`, `vestingDuration`, `recipients`, `amounts`
- **Integration**: `integrator`, `liquidityMigratorData`

## Validation Rules

- Name and symbol are required and non-empty
- Total supply and tokens to sell must be positive
- Tick range must be valid (startTick < endTick)
- Duration and epoch length must be positive
- Tick spacing must be positive and divide gamma evenly
- Epoch length must divide total duration evenly

## Workflow Optimization

The factory automatically:

- Mines optimal hook addresses with required flags
- Validates parameter compatibility before deployment
- Provides gas estimation through simulation
- Optimizes gamma calculation for efficient price discovery
