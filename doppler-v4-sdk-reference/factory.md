# Factory

The `ReadWriteFactory` class provides comprehensive read and write operations for the Doppler V4 airlock contract. It extends `ReadFactory` with capabilities for creating pools, mining hook addresses, and migrating assets.

## Key Features

- **Pool Creation**: Deploy complete Doppler ecosystems with tokens, hooks, and governance
- **Hook Mining**: Find optimal hook addresses with required flags
- **Asset Migration**: Move liquidity from price discovery to standard trading
- **Parameter Validation**: Automatic validation and optimization of deployment parameters
- **Gamma Calculation**: Compute optimal price movement parameters

## Constructor

```typescript
new ReadWriteFactory(address: Address, drift: Drift<ReadWriteAdapter>)
```

**Parameters:**
- `address` - The address of the factory contract
- `drift` - A Drift instance with read-write adapter capabilities

## Core Methods

### buildConfig

Builds complete configuration for creating a new Doppler pool. This method validates parameters, converts price ranges to tick ranges, computes optimal gamma, mines hook addresses, and encodes factory data.

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

## Utility Methods

### encodeCustomLPLiquidityMigratorData

Encodes custom LP liquidity migrator data for specialized migration scenarios.

```typescript
encodeCustomLPLiquidityMigratorData(customLPConfig: {
  customLPWad: bigint;
  customLPRecipient: Address;
  lockupPeriod: number;
}): Hex
```

**Parameters:**
- `customLPConfig` - Configuration for custom LP migration

**Returns:** Encoded migrator data

## Example Usage

```typescript
import { ReadWriteFactory } from '@delvtech/drift';

// Create factory instance
const factory = new ReadWriteFactory(airlockAddress, drift);

// Build complete configuration
const { createParams, hook, token } = factory.buildConfig({
  name: "Community Token",
  symbol: "COMM",
  totalSupply: parseEther("1000000"),
  numTokensToSell: parseEther("500000"),
  tickRange: { startTick: -276320, endTick: -230260 },
  duration: 30, // 30 days
  epochLength: 3600, // 1 hour epochs
  tickSpacing: 60,
  fee: 3000,
  minProceeds: parseEther("1000"),
  maxProceeds: parseEther("10000"),
  blockTimestamp: Math.floor(Date.now() / 1000),
  yearlyMintRate: parseEther("100000"),
  vestingDuration: BigInt(365 * 24 * 3600), // 1 year
  recipients: [recipient1, recipient2],
  amounts: [parseEther("50000"), parseEther("25000")],
  tokenURI: "https://example.com/token.json",
  integrator: zeroAddress
}, addresses);

// Simulate creation to estimate gas
const simulation = await factory.simulateCreate(createParams);
console.log(`Estimated gas: ${simulation.request.gas}`);

// Create the pool
const txHash = await factory.create(createParams, {
  gasLimit: 5000000n
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

## Gas Optimization

The factory automatically:
- Mines optimal hook addresses with required flags
- Validates parameter compatibility before deployment
- Provides gas estimation through simulation
- Optimizes gamma calculation for efficient price discovery