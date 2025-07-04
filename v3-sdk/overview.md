---
description: Overview of the Doppler V3 SDK
icon: clipboard-question
---

# Overview

## Doppler V3 SDK Overview

### What is the Doppler V3 SDK?

The Doppler V3 SDK is a TypeScript library that provides developers with comprehensive tools to interact with the Doppler protocol's V3 implementation. It enables seamless integration with Doppler's token creation, price discovery, and liquidity migration systems across multiple EVM-compatible networks.

The SDK abstracts the complexity of smart contract interactions, providing type-safe methods for creating tokens, managing pools, and handling the complete lifecycle of assets on the Doppler protocol. It's designed to support both read-only operations for data querying and write operations for token deployment and pool management.

### What is Drift?

The Doppler V3 SDK is built on top of [Drift](https://delvtech.github.io/drift/), a modern blockchain interaction framework that simplifies Web3 development by:

* Providing unified interfaces for different wallet providers and RPC endpoints
* Handling transaction lifecycle management with automatic retry logic
* Offering type-safe contract interaction patterns
* Supporting both read and write operations with consistent error handling

### Core Concepts

#### Supported Networks

The V3 SDK supports multiple networks where Doppler protocol contracts are deployed:

| Network              | Chain ID | Environment | Purpose                       |
| -------------------- | -------- | ----------- | ----------------------------- |
| **Base Sepolia**     | `84532`  | Testnet     | Development and testing       |
| **Base Mainnet**     | `8453`   | Production  | Live token deployments        |
| **Unichain Mainnet** | `130`    | Production  | Multi-chain token deployments |
| **Unichain Sepolia** | `1301`   | Testnet     | Development and testing       |
| **Ink**              | `57073`  | Production  | Specialized chain deployments |

Network addresses and configurations are automatically managed through the `DOPPLER_V3_ADDRESSES` constant, making it easy to switch between networks.

#### Factory Classes

The SDK provides two main factory classes for different interaction patterns:

| Class              | Purpose                              | Use Cases                           |
| ------------------ | ------------------------------------ | ----------------------------------- |
| `ReadFactory`      | Query protocol state and read data   | Analytics, monitoring, data display |
| `ReadWriteFactory` | Create tokens and manage deployments | Token creation, pool management     |

#### Asset Lifecycle Management

The SDK handles the complete Doppler asset lifecycle:

1. **Token Creation**: Deploy new DERC20 tokens with customizable parameters
2. **Price Discovery**: Manage initial liquidity and price discovery phases
3. **Migration**: Handle liquidity migration to Uniswap V2 pools (or V4 with fee streaming)
4. **Pool Operations**: Interact with both price discovery and migrated pools

### Key Features

#### Token Creation & Configuration

The SDK provides comprehensive token creation capabilities:

* **Flexible Configuration**: Support for custom token parameters, sale configurations, and governance settings
* **Default Templates**: Pre-configured templates for common use cases with `defaultSaleConfig`, `defaultV3PoolConfig`, and `defaultGovernanceConfig`
* **Parameter Validation**: Automatic validation of creation parameters to prevent deployment errors
* **Gas Estimation**: Built-in simulation capabilities for accurate gas estimation

#### Pool Interaction

Comprehensive pool management and querying:

* **Pool State Queries**: Access to slot0, token information, and fee structures
* **Event Tracking**: Retrieve mint, burn, and swap events from pools
* **Multi-Protocol Support**: Handle both V3 price discovery pools and migrated V2/V4 pools
* **Real-time Data**: Live access to pool liquidity, prices, and trading activity

#### V4 Migration Support

Built-in support for migrating to Uniswap V4 with advanced features:

* **Fee Streaming Configuration**: Set up fee distribution to multiple beneficiaries
* **Beneficiary Management**: Automatic sorting and validation of beneficiary data
* **Migration Validation**: Ensure proper configuration before migration execution

### Contract Integration

#### Core Contracts

The SDK integrates with several key contract types:

| Contract Type       | Purpose                         | SDK Integration                    |
| ------------------- | ------------------------------- | ---------------------------------- |
| `Airlock`           | Main factory for token creation | `ReadFactory`, `ReadWriteFactory`  |
| `TokenFactory`      | DERC20 token deployment         | Automatic integration via factory  |
| `GovernanceFactory` | Governance contract creation    | Configurable through creation flow |
| `PoolInitializer`   | V3 pool initialization          | Automatic pool setup               |
| `LiquidityMigrator` | Handle pool migrations          | Migration flow management          |
| `UniswapV3Pool`     | Price discovery pools           | `ReadUniswapV3Pool` class          |

#### Event Handling

The SDK provides access to important protocol events:

* **Create Events**: Track new token deployments with full asset data
* **Migrate Events**: Monitor liquidity migrations between pool types
* **Pool Events**: Access mint, burn, and swap events from individual pools
* **Transfer Events**: Track token transfers and balance changes

### Configuration & Customization

#### Default Configurations

The SDK exports several default configurations that can be customized:

```typescript
// Available default configurations
import {
  defaultSaleConfig,
  defaultV3PoolConfig,
  defaultGovernanceConfig,
  defaultVestingConfig
} from 'doppler-v3-sdk';
```

#### Custom Parameters

Key customizable parameters include:

* **Sale Configuration**: Token sale parameters, pricing, and distribution
* **Pool Configuration**: Fee tiers, tick spacing, and initial liquidity
* **Governance Configuration**: Voting parameters, proposal thresholds, and timelock settings
* **Vesting Configuration**: Token vesting schedules and inflation parameters

### Error Handling & Validation

#### Built-in Validations

The SDK includes comprehensive validation for:

* **Parameter Consistency**: Ensure all configuration parameters are compatible
* **Network Compatibility**: Validate contract addresses for the target network
* **Mathematical Constraints**: Check that numerical parameters meet protocol requirements
* **Beneficiary Configuration**: Validate fee streaming beneficiary shares sum to 100%

#### Error Types

Common error scenarios handled by the SDK:

* **Configuration Errors**: Invalid parameter combinations or missing required fields
* **Network Errors**: RPC failures, transaction timeouts, and connection issues
* **Contract Errors**: Revert reasons, gas estimation failures, and execution errors
* **Validation Errors**: Parameter validation failures and constraint violations

## Next steps

For detailed implementation examples, see the [Getting Started Guide](getting-started.md) and explore the comprehensive API documentation in the [Factory Reference](factory.md).
