---
description: Getting Started with Doppler V3 SDK
icon: rocket
---

# Get Started

This section guides you through setting up and using the Doppler V3 SDK to interact with the Doppler protocol.

## Prerequisites

* **Node.js**: Version `18.14` or higher
* **npm** or **yarn**: Package manager for installing dependencies
* **Web3 Provider**: Access to Ethereum RPC endpoints (Infura, Alchemy, etc.)
* **Wallet**: MetaMask or similar wallet for transaction signing

## Installation

Install the Doppler V3 SDK package:

```bash
npm install doppler-v3-sdk
# or
yarn add doppler-v3-sdk
```

## Basic Setup

### 1. Import the SDK

```typescript
import { 
  ReadFactory, 
  ReadWriteFactory, 
  ReadUniswapV3Pool,
  DOPPLER_V3_ADDRESSES 
} from 'doppler-v3-sdk';
```

### 2. Initialize Drift Client

The SDK uses Drift for blockchain interactions. Set up your Drift client with both read and write capabilities:

```typescript
import { Drift } from 'drift';

// For read-only operations
const drift = new Drift({
  rpcUrl: 'https://sepolia.base.org',
  chainId: 84532 // Base Sepolia
});

// For read-write operations (with wallet)
const driftWithWallet = new Drift({
  rpcUrl: 'https://sepolia.base.org',
  chainId: 84532,
  wallet: yourWalletProvider // MetaMask, WalletConnect, etc.
});
```

### 3. Get Protocol Addresses

```typescript
// Get addresses for the current network
const chainId = 84532; // Base Sepolia (change to 8453 for Base Mainnet, 130 for Unichain Mainnet, etc.)
const addresses = DOPPLER_V3_ADDRESSES[chainId];
const airlockAddress = addresses.airlock;
```

## Core Concepts

### Factory Classes

The SDK provides two main factory classes:

* **`ReadFactory`**: For querying protocol state and reading data
* **`ReadWriteFactory`**: For creating tokens and interacting with pools

### Asset Lifecycle

1. **Token Creation**: Deploy new tokens through the factory
2. **Price Discovery**: Initial liquidity provision and price discovery phase
3. **Migration**: Move liquidity to standard Uniswap V2 pools (or V4 pools if using fee streaming)
4. **Trading**: Normal trading on migrated pools

## Quick Start Examples

### Reading Protocol Data

```typescript
// Create a read factory instance
const factory = new ReadFactory(airlockAddress, drift);

// Get information about a deployed asset
const assetData = await factory.getAssetData(tokenAddress);
console.log('Asset details:', {
  numeraire: assetData.numeraire,
  governance: assetData.governance,
  pool: assetData.pool,
  totalSupply: assetData.totalSupply.toString()
});

// Check module state
const moduleState = await factory.getModuleState(moduleAddress);
console.log('Module state:', moduleState);
```

### Creating a New Token

```typescript
// Create a read-write factory instance
const factory = new ReadWriteFactory(airlockAddress, driftWithWallet);

// Define token creation parameters
const createParams = {
  integrator: integratorAddress,
  userAddress: userAddress,
  numeraire: numeraireAddress, // USDC, WETH, etc.
  contracts: {
    tokenFactory: addresses.tokenFactory,
    governanceFactory: addresses.governanceFactory,
    poolInitializer: addresses.poolInitializer,
    liquidityMigrator: addresses.liquidityMigrator
  },
  tokenConfig: {
    name: "My Token",
    symbol: "MTK",
    decimals: 18
  },
  vestingConfig: "default" // or custom configuration
};

// Encode the creation parameters
const { createParams: encodedParams } = factory.encode(createParams);

// Simulate the creation transaction
const simulation = await factory.simulateCreate(encodedParams);
console.log('Gas estimate:', simulation.gasEstimate);

// Execute the creation transaction
const txHash = await factory.create(encodedParams);
console.log('Transaction hash:', txHash);

// Wait for transaction confirmation and get the deployed asset address
const receipt = await drift.waitForTransactionReceipt({ hash: txHash });
const createEvent = receipt.logs.find(log => 
  log.topics[0] === '0x...' // Create event signature
);
const deployedTokenAddress = `0x${createEvent.topics[1].slice(26)}`;
console.log('Deployed token address:', deployedTokenAddress);
```

### Interacting with Your Created Pool

Once you've created a token, you can interact with its price discovery pool:

```typescript
// Get the asset data to find the price discovery pool address
const assetData = await factory.getAssetData(deployedTokenAddress);
const poolAddress = assetData.pool;

// Create a pool instance for the price discovery pool
const pool = new ReadUniswapV3Pool(poolAddress, drift);

// Get pool information
const slot0 = await pool.getSlot0();
const token0 = await pool.getToken0();
const token1 = await pool.getToken1();
const fee = await pool.getFee();

console.log('Price discovery pool info:', {
  sqrtPriceX96: slot0.sqrtPriceX96.toString(),
  tick: slot0.tick,
  token0,
  token1,
  fee
});

// Get pool events (will show the initial mint from token creation)
const mintEvents = await pool.getMintEvents();
const swapEvents = await pool.getSwapEvents();
console.log(`Found ${mintEvents.length} mint events and ${swapEvents.length} swap events`);

// Note: After migration, liquidity moves to a V2 pool (or V4 if using fee streaming)
// The migrationPool address can be found at assetData.migrationPool
```

## Environment Configuration

### Required Environment Variables

```bash
# RPC Endpoint
RPC_URL="https://sepolia.base.org"

# Wallet Private Key (for automated transactions)
PRIVATE_KEY="your-private-key"

# Network Configuration
CHAIN_ID=84532
```

### Network Support

The SDK supports multiple networks:

* **Base Sepolia** (chainId: 84532) - Testnet
* **Base Mainnet** (chainId: 8453) - Production
* **Unichain Mainnet** (chainId: 130) - Production
* **Unichain Sepolia** (chainId: 1301) - Testnet
* **Ink** (chainId: 57073) - Production

For complete network addresses and additional supported networks, see the [Contract Addresses](../resources/contract-addresses.md) documentation.

You can get free Base Sepolia ETH from the [Base Sepolia faucet](https://docs.base.org/tools/network-faucets) to test your applications.

## Error Handling

The SDK provides comprehensive error handling for common scenarios:

```typescript
try {
  const assetData = await factory.getAssetData(tokenAddress);
} catch (error) {
  if (error.message.includes('Asset not found')) {
    console.log('Token not deployed on Doppler');
  } else {
    console.error('Unexpected error:', error);
  }
}
```

## Next Steps

* Explore the [Factory Reference](factory.md) for detailed API documentation
* Learn about [Token Operations](token.md) for managing deployed tokens
* Understand [Quoter Usage](quoter.md) for price calculations
* Check out [V4 Migration](v4-migrator.md) for upgrading to V4

## Support

For additional help and examples:

* Check the [Token Launch Examples](../v4-sdk/token-launch-examples.md) for comprehensive deployment scenarios
* Review the [Implementation Guide](../how-it-works/implementation.md) for protocol details
* Join the community for discussions and support
