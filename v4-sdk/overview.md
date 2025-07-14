---
description: Overview of the Doppler V4 SDK
icon: clipboard-question
---

# Overview

## Doppler V4 SDK Overview

### What is the Doppler V4 SDK?

The Doppler V4 SDK is a next-generation TypeScript library that provides developers with advanced tools to interact with the Doppler protocol's V4 implementation. Built on the foundation of Uniswap V4's hook architecture, it enables sophisticated token creation with dynamic bonding curves, advanced pool management, and comprehensive analytics capabilities across multiple EVM-compatible networks.

### What is Drift?

The Doppler V4 SDK is built on top of [Drift](https://delvtech.github.io/drift/), a modern blockchain interaction framework that simplifies Web3 development by:

* Providing unified interfaces for different wallet providers and RPC endpoints
* Handling transaction lifecycle management with automatic retry logic
* Offering type-safe contract interaction patterns
* Supporting both read and write operations with consistent error handling

### Core Concepts

#### Supported Networks

The V4 SDK supports multiple production and testing networks:

| Network              | Chain ID | Environment | Purpose                       |
| -------------------- | -------- | ----------- | ----------------------------- |
| **Base Mainnet**     | `8453`   | Production  | Live token deployments        |
| **Unichain Mainnet** | `130`    | Production  | Live token deployments        |
| **Ink**              | `57073`  | Production  | Live token deployments        |
| **Base Sepolia**     | `84532`  | Testnet     | Development and testing       |

Network configurations are managed through `DOPPLER_V4_ADDRESSES`, providing seamless network switching and automatic contract address resolution.

#### Factory Architecture

The V4 SDK provides specialized factory classes optimized for advanced operations:

| Class              | Purpose                                   | Key Features                             |
| ------------------ | ----------------------------------------- | ---------------------------------------- |
| `ReadFactory`      | Query protocol state and analytics data   | Asset data, module states, events        |
| `ReadWriteFactory` | Deploy tokens with hooks and manage pools | Hook mining, config building, deployment |

#### Advanced Asset Lifecycle

The V4 SDK manages sophisticated asset lifecycles with enhanced capabilities:

1. **Token Creation**: Deploy DERC20 tokens with custom hooks and dynamic bonding curves
2. **Price Discovery**: Advanced price discovery with hook-based customization
3. **Trading**: Enhanced trading with V4 features and custom logic

### Key Features

#### Dynamic Bonding Curves

Revolutionary approach to token pricing and liquidity:

* **Custom Hook Integration**: Deploy tokens with specialized hooks that implement dynamic bonding curves
* **Adaptive Pricing**: Hooks can adjust pricing based on market conditions, trading volume, and other parameters
* **Flexible Logic**: Support for complex mathematical models and custom trading algorithms

#### Advanced Configuration System

Comprehensive configuration management for complex deployments:

* [**Pre-deployment Configuration**](#doppler-predeployment-configuration-parameters): `DopplerPreDeploymentConfig` for complete setup specification
* **Hook Mining**: Automatic discovery of optimal hook addresses with required flags
* **Parameter Validation**: Advanced validation ensuring configuration consistency and mathematical correctness
* **Gas Optimization**: Intelligent gas estimation and optimization for complex transactions

#### Doppler Lens Integration

Powerful analytics and data querying system through the DopplerLensQuoter:

* **Pool State Queries**: Real-time pool price, tick, and liquidity information
* **Virtual Position Data**: Access to combined liquidity across all position types
* **Simulation-Based**: Gas-free state queries using revert-based simulation
* **Price Discovery Support**: Essential for monitoring active Dutch auctions

### Contract Integration

#### Core V4 Contracts

The SDK integrates with advanced V4 contract architecture:

| Contract Type       | Purpose                             | V4 Enhancements                           |
| ------------------- | ----------------------------------- | ----------------------------------------- |
| `Airlock`           | Advanced factory for token creation | Hook support, complex configurations      |
| `TokenFactory`      | V4DERC20 token deployment           | Hook integration, dynamic features        |
| `GovernanceFactory` | Enhanced governance systems         | Advanced voting mechanisms                |
| `PoolInitializer`   | V4 pool initialization with hooks   | Hook mining, custom logic setup           |
| `DopplerLensQuoter` | Advanced data querying system       | Pool state queries, virtual position data |
| `UniswapV4Pool`     | Hook-enabled trading pools          | Custom trading logic, dynamic fees        |

#### Hook System

Revolutionary hook-based architecture for custom trading logic:

* **Hook Flags**: Configure which lifecycle events trigger custom logic
* **Address Mining**: Automatic discovery of valid hook addresses with required properties
* **Custom Logic**: Support for complex trading algorithms, dynamic fees, and custom behaviors
* **Validation**: Comprehensive validation of hook configurations and compatibility

### Advanced Capabilities

#### Configuration Building

Sophisticated configuration management system:

```typescript
// Advanced configuration building
const preDeploymentConfig: DopplerPreDeploymentConfig = { ... };
const { createParams, hook, token } = factory.buildConfig(
  preDeploymentConfig, 
  addresses
);
```

Key capabilities include:

* **Parameter Optimization**: Automatic optimization of configuration parameters
* **Compatibility Checking**: Validation of parameter combinations and constraints
* **Hook Integration**: Seamless integration of custom hooks with token logic
* **Gas Estimation**: Accurate gas estimation for complex deployments

#### Doppler Predeployment Configuration Parameters

| Key                           | Type                                                     | Purpose                                                                 |
| ----------------------------- | -----------------------------------------------------    | ----------------------------------------------------------------------- |
| `name`                        | string                                                   | Name of the token being deployed                                        |
| `symbol`                      | string                                                   | Symbol representing the token                                           |
| `totalSupply`                 | bigint                                                   | Total supply of tokens available                                        |
| `numTokensToSell`             | bigint                                                   | Amount of tokens to sell                                                |
| `tokenURI`                    | string                                                   | URI pointing to the token's metadata                                    |
| `blockTimestamp`              | number                                                   | Timestamp of the block when the configuration is set                    |
| `startTimeOffset`             | number                                                   | Offset in **days** from the current time to the start of the sale       |
| `duration`                    | number                                                   | Duration of the sale in **days**                                        |
| `epochLength`                 | number                                                   | Length of each epoch in **seconds**                                     |
| `numeraire`                   | Address **(optional)**                                   | Address of the numéraire token, defaults to native if not provided      |
| `tickRange` (TickRange)       | { startTick: number, endTick: number } **(optional)**    | Range of ticks for price adjustments                                    |
| `priceRange` (PriceRange)     | { startPrice: number, endPrice: number } **(optional)**  | Range of prices for the token                                           |
| `tickSpacing`                 | number                                                   | Spacing between ticks for price adjustments                             |
| `gamma`                       | number **(optional)**                                    | Gamma value for price calculations                                      |
| `fee`                         | number                                                   | Transaction fee in basis points (bips)                                  |
| `minProceeds`                 | bigint                                                   | Minimum range for auction target                                        |
| `maxProceeds`                 | bigint                                                   | Maximum range for auction target                                        |
| `numPdSlugs`                  | number **(optional)**                                    | Number of price discovery slugs                                         |
| `yearlyMintRate`              | bigint                                                   | Rate at which tokens are minted annually                                |
| `vestingDuration`             | bigint                                                   | Duration of the vesting period in seconds                               |
| `recipients`                  | Address[]                                                | List of addresses receiving vested tokens                               |
| `amounts`                     | bigint[]                                                 | Amounts of tokens allocated to each recipient                           |
| `liquidityMigratorData`       | Hex **(optional)**                                       | Encoded data for liquidity migration                                    |
| `integrator`                  | Address                                                  | Address of the integrator managing the deployment                       |

#### Data Analytics

Comprehensive analytics through the DopplerLensQuoter system:

* **Pool State**: Real-time price, tick, and total liquidity information
* **Virtual Positions**: Combined token amounts across all position types
* **Simulation Queries**: Gas-free state access using revert-based simulation
* **Price Discovery**: Real-time monitoring of Dutch auction states

#### Error Handling & Validation

Advanced error handling for complex operations:

* **Configuration Validation**: Comprehensive validation of complex parameter sets
* **Hook Validation**: Verification of hook logic and compatibility
* **Network Resilience**: Robust handling of network issues and transaction failures
* **Gas Management**: Intelligent gas estimation and optimization

## Next steps

For detailed implementation guidance, explore the [Getting Started Guide](getting-started.md), [Factory Reference](factory.md), and [Doppler Lens Documentation](lens.md) for comprehensive API coverage.
