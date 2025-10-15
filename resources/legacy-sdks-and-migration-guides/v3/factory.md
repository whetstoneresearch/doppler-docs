---
description: Factory Class Reference
icon: square-terminal
---

# Factory

### Factory Overview

The factory is the main entrypoint for interacting with and creating tokens on the Doppler protocol. It is responsible for deploying new tokens, parameterizing pools, migrating existing tokens, and serving information about the state of the protocol.

#### ReadFactory

Read-only operations for the Doppler factory.

```typescript
const factory = new ReadFactory(address: Address, drift: Drift<ReadAdapter>);
```

Methods:

* `getModuleState(address: Address): Promise<ModuleState>`
  * Returns the state of a module (NotWhitelisted, TokenFactory, GovernanceFactory, HookFactory, Migrator)
* `getAssetData(asset: Address): Promise<AssetData>`
  *   Returns information about a DERC20 token deployed by the airlock contract:

      ```typescript
      interface AssetData {
        numeraire: Address;
        timelock: Address;
        governance: Address;
        liquidityMigrator: Address;
        poolInitializer: Address;
        pool: Address;
        migrationPool: Address;
        numTokensToSell: bigint;
        totalSupply: bigint;
        integrator: Address;
      }
      ```
* `getCreateEvents(): Promise<Event[]>`
* `getMigrateEvents(): Promise<Event[]>`

#### ReadWriteFactory

Extends ReadFactory with write operations.

The primary use case for the ReadWriteFactory is to encode the parameters for, simulate, and invoke the `create` method. Proper utilization of the ReadWriteFactory involves the following steps:

1. Initialize a new `ReadWriteFactory` with the factory (airlock) address and a drift client with both a public and wallet client attached.
2. Pass `CreateV3PoolParams` to the `encode` method in order to build the calldata for the `create` method. Note that the `saleConfig`, `v3PoolConfig`, and `governanceConfig` are optional and can be omitted. Omission of these parameters will use the default values for each respective config. The default values are exported from the `doppler-v3-sdk` package under `defaultSaleConfig`, `defaultV3PoolConfig`, and `defaultGovernanceConfig`, and `defaultVestingConfig`. These default configs can be modified to customize the token creation process, but modifications should be made with caution as they may break some of the algebraic invariants of the Doppler protocol, or result in unexpected outcomes. For example, custom implementations should note that vesting is defined as a percentage of inflation relative to total token supply.

```typescript
export interface CreateV3PoolParams {
  integrator: Address;
  userAddress: Address;
  numeraire: Address;
  contracts: InitializerContractDependencies;
  tokenConfig: TokenConfig;
  saleConfig?: Partial<SaleConfig>;
  v3PoolConfig?: Partial<V3PoolConfig>;
  vestingConfig: VestingConfig | "default";
  governanceConfig?: Partial<GovernanceConfig>;
}

public encode(params: CreateV3PoolParams): {
    createParams: CreateParams;
    v3PoolConfig: V3PoolConfig;
}
```

`encode` returns the following payload, which is used to simulate and invoke the `create` method:

```typescript
interface CreateParams {
  initialSupply: bigint;
  numTokensToSell: bigint;
  numeraire: Address;
  tokenFactory: Address;
  tokenFactoryData: Hex;
  governanceFactory: Address;
  governanceFactoryData: Hex;
  poolInitializer: Address;
  poolInitializerData: Hex;
  liquidityMigrator: Address;
  liquidityMigratorData: Hex;
  integrator: Address;
  salt: Hex;
}
```

3. Simulate the `create` method to ensure the parameters are correct

```typescript
  public async simulateCreate(params: CreateParams): Promise<FunctionReturn<AirlockABI, "create">>
```

**Note:** The `simulateCreate` method is a simulation of the `create` method and does not consume any gas. It is recommended to simulate the `create` method before invoking it in order to ensure that the parameters are correct and that the method will succeed.

4. Invoke the `create` method

### Streamable V3 (Lockable V3 Initializer)

The SDK supports creating V3 pools with fee streaming capabilities through the lockable V3 initializer. When beneficiaries are specified in the `v3PoolConfig`, the pool will be permanently locked and stream trading fees to the specified beneficiaries.

To use the lockable V3 initializer:

1. **Specify the lockable initializer address** in your contracts configuration:

```typescript
const contracts = {
  tokenFactory: DOPPLER_V3_ADDRESSES[chainId].tokenFactory,
  governanceFactory: DOPPLER_V3_ADDRESSES[chainId].governanceFactory,
  v3Initializer: DOPPLER_V3_ADDRESSES[chainId].lockableV3Initializer, // Use lockable initializer
  liquidityMigrator: DOPPLER_V3_ADDRESSES[chainId].liquidityMigrator,
};
```

2. **Add beneficiaries to the v3PoolConfig**:

```typescript
const v3PoolConfig = {
  // ... standard pool config
  beneficiaries: [
    {
      beneficiary: ownerAddress, // Airlock owner must receive exactly 5%
      shares: parseEther("0.05")
    },
    {
      beneficiary: projectAddress,
      shares: parseEther("0.95") 
    }
  ]
};
```

**Important requirements:**

* Total beneficiary shares must equal 1e18 (100%)
* The Airlock owner must be included with exactly 5% shares
* Beneficiaries must be sorted by address in ascending order
* Pools with beneficiaries will be permanently locked and never migrate

For more detailed information about streamable V3 pools, see the [Streamable V3 documentation](../../../v3-sdk/streamable-v3.md).

### V4 Migrator Support

The ReadWriteFactory now includes helper functions for configuring V4 migration with fee streaming:

* `sortBeneficiaries(beneficiaries: BeneficiaryData[]): BeneficiaryData[]`
  * Sorts beneficiaries by address in ascending order (required by the V4 migrator contract)
* `encodeV4MigratorData(data: V4MigratorData): Hex`
  * Encodes V4 migrator configuration including fee tier, tick spacing, lock duration, and beneficiaries
  * Validates that beneficiaries are properly sorted and shares sum to exactly 1e18 (100%)

For detailed V4 migrator usage, see the [V4 Migrator documentation](custom-fees.md).

```typescript
  public async create(
    params: CreateParams,
    options?: ContractWriteOptions & OnMinedParam
  ): Promise<Hex>
```

#### Pool Operations

**ReadUniswapV3Pool**

Read operations for Doppler V3 pools.

```typescript
const pool = new ReadUniswapV3Pool(address: Address, drift?: Drift<ReadAdapter>);
```

Methods:

* `getMintEvents(): Promise<Event[]>`
* `getBurnEvents(): Promise<Event[]>`
* `getSwapEvents(): Promise<Event[]>`
* `getSlot0(): Promise<{sqrtPriceX96: bigint, tick: number}>`
* `getToken0(): Promise<Address>`
* `getToken1(): Promise<Address>`
* `getFee(): Promise<number>`

#### Event Types

**Factory Events**

```typescript
interface CreateEvent {
  asset: Address;
  numeraire: Address;
  initializer: Address;
  poolOrHook: Address;
}

interface MigrateEvent {
  asset: Address;
  pool: Address;
}
```

**Pool Events**

```typescript
interface MintEvent {
  owner: Address;
  tickLower: number;
  tickUpper: number;
  amount: bigint;
  amount0: bigint;
  amount1: bigint;
}

interface BurnEvent {
  owner: Address;
  tickLower: number;
  tickUpper: number;
  amount: bigint;
  amount0: bigint;
  amount1: bigint;
}

interface SwapEvent {
  sender: Address;
  recipient: Address;
  amount0: bigint;
  amount1: bigint;
  sqrtPriceX96: bigint;
  liquidity: bigint;
  tick: number;
}
```

#### Network Configuration

```typescript
const DOPPLER_V3_ADDRESSES: { [chainId: number]: DopplerV3Addresses };
```

Supported Networks:

* Unichain Sepolia (chainId: 1301)
* Unichain (chainId: 130)

### Doppler v4 SDK API Reference

Coming Soon!
