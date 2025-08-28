---
description: Getting Started with Doppler V4 SDK
icon: rocket
---

# Get Started

This section guides you through setting up and using the Doppler V4 SDK to interact with the latest version of the Doppler protocol.

## Prerequisites

* **Node.js**: Version `18.14` or higher
* **npm** or **yarn**: Package manager for installing dependencies
* **Web3 Provider**: Access to Ethereum RPC endpoints (Infura, Alchemy, etc.)
* **Wallet**: MetaMask or similar wallet for transaction signing

## Installation

Install the Doppler V4 SDK, Viem, and Drift packages:

```bash
npm install doppler-v4-sdk viem @delvtech/drift @delvtech/drift-viem
# or
yarn add doppler-v4-sdk viem @delvtech/drift @delvtech/drift-viem
```

The SDK uses [Drift](https://github.com/delvtech/drift) for blockchain interactions.

## Required Environment Variables

```bash
# RPC Endpoint
RPC_URL="https://sepolia.base.org"

# Wallet Private Key (for automated transactions)
PRIVATE_KEY="your-private-key"

# Network Configuration
CHAIN_ID=84532
```

## Basic Setup

### 1. Import the SDK

```typescript
import { 
  ReadFactory, 
  ReadWriteFactory, 
  Lens,
  DOPPLER_V4_ADDRESSES 
} from 'doppler-v4-sdk';
```

### 2. Initialize Drift Client

Set up your Drift client with both read and write capabilities:

#### Read-only operations

```typescript
import { Drift } from 'drift';

// For read-only operations
const drift = new Drift({
  rpcUrl: 'https://sepolia.base.org',
  chainId: 84532 // Base Sepolia
});
```

#### Read-write operations

```typescript
import { createDrift } from "@delvtech/drift";
import { createPublicClient, createWalletClient, http, PublicClient } from "viem";

const publicClient = createPublicClient({
  chain: baseSepolia,
  transport: http('https://sepolia.base.org'),
});

const walletClient = createWalletClient({
  chain: baseSepolia,
  transport: http('https://sepolia.base.org'),
  account: privateKeyToAccount(WALLET_PRIVATE_KEY),
});

// For read-write operations (with wallet)
const driftWithWallet = createDrift({
  adapter: viemAdapter({ publicClient, walletClient }),
});
```

### 3. Get Protocol Addresses

```typescript
// Get addresses for the current network
const chainId = 84532; // Base Sepolia (change to 8453 for Base Mainnet, 130 for Unichain Mainnet, etc.)
const addresses = DOPPLER_V4_ADDRESSES[chainId];
const airlockAddress = addresses.airlock;
```

## Core Concepts

### Factory Classes

The V4 SDK provides two main factory classes:

* **`ReadFactory`**: For querying protocol state and reading data
* **`ReadWriteFactory`**: For creating tokens with hooks

### Key V4 Features

* **Dynamic bonding curves**: Custom hooks supporting dynamic bonding curves

### Asset Lifecycle

1. **Token Creation**: Deploy new tokens with custom hooks
2. **Price Discovery**: Initial liquidity provision and price discovery phase
3. **Trading**: Enhanced trading with V4 features

## Quick Start Examples

### Reading Protocol Data

```typescript
// Create a read factory instance
const factory = new ReadFactory(
  airlockAddress,
  drift,
);

// Get information about a deployed asset
const assetData = await factory.getAssetData(tokenAddress);
console.log('Asset details:', {
  numeraire: assetData.numeraire,
  governance: assetData.governance,
  pool: assetData.pool,
  migrationPool: assetData.migrationPool,
  totalSupply: assetData.totalSupply.toString()
});

// Check module state
const moduleState = await factory.getModuleState(moduleAddress);
console.log('Module state:', moduleState);
```

### Creating a New Token with Hooks

```typescript
const addresses = DOPPLER_V4_ADDRESSES[chainId];
const airlockAddress = addresses.airlock;
const bundlerAddress = addresses.bundler;
const tokenURI = "https://example.com/token-metadata.json";
const integrator: Address = "0x...";

// Create a read-write factory instance
const factory = new ReadWriteFactory(
  airlockAddress,
  bundlerAddress,
  driftWithWallet
);

// Define pre-deployment configuration
const preDeploymentConfig: DopplerPreDeploymentConfig = {
  name: tokenName,
  symbol: tokenSymbol,
  totalSupply: parseEther('1_000_000_000'),
  numTokensToSell: parseEther('600_000_000'),
  tokenURI,
  blockTimestamp: Math.floor(Date.now() / 1000),
  startTimeOffset: 1,
  duration: 1 / 4,
  epochLength: 200,
  gamma: 800,
  tickRange: {
    startTick: 174_312,
    endTick: 186_840,
  },
  tickSpacing: 2,
  fee: 20_000, // 2%
  minProceeds: parseEther('2'),
  maxProceeds: parseEther('4'),
  yearlyMintRate: 0n,
  vestingDuration: BigInt(24 * 60 * 60 * 365), // Seconds in a year
  recipients: [wallet.account.address],
  amounts: [parseEther('50_000_000')],
  numPdSlugs: 15,
  integrator,
};

// Build the complete configuration
const { createParams, hook, token } = factory.buildConfig(
  preDeploymentConfig, 
  addresses
);

// Simulate the creation transaction
const simulation = await factory.simulateCreate(createParams);
console.log('Gas estimate:', simulation.gasEstimate);

// Execute the creation transaction
const txHash = await factory.create(createParams);
console.log('Transaction hash:', txHash);

// Wait for transaction confirmation and get the deployed asset address
const receipt = await drift.waitForTransactionReceipt({ hash: txHash });
const createEvent = receipt.logs.find(log => 
  log.topics[0] === '0x...' // Create event signature
);
const deployedTokenAddress = `0x${createEvent.topics[1].slice(26)}`;
console.log('Deployed token address:', deployedTokenAddress);
```

### Using the Doppler Lens for Advanced Queries

After creating your token, you can use the DopplerLensQuoter to get detailed information about your pool:

```typescript
// Get the asset data to find the pool information
const assetData = await factory.getAssetData(deployedTokenAddress);

// Create a doppler lens instance
const lens = new ReadDopplerLens(addresses.dopplerLensQuoter, drift);

// Define pool key for the created token
const poolKey = {
  currency0: assetData.numeraire,
  currency1: deployedTokenAddress,
  fee: 20_000, // 2% fee tier
  tickSpacing: 2,
  hooks: assetData.hook // Hook address from asset data
};

// Get comprehensive pool state data
const poolData = await lens.quoteDopplerLensData({
  poolKey,
  zeroForOne: true,
  exactAmount: 1n, // Minimal amount for state query
  hookData: "0x"
});

console.log('Pool state:', {
  sqrtPriceX96: poolData.sqrtPriceX96.toString(),
  tick: poolData.tick,
  totalToken0: poolData.amount0.toString(),
  totalToken1: poolData.amount1.toString()
});
```

### Network Support

The V4 SDK supports multiple networks:

* **Base Sepolia** (chainId: 84532) - Testnet
* **Base Mainnet** (chainId: 8453) - Production
* **Unichain Mainnet** (chainId: 130) - Production
* **Unichain Sepolia** (chainId: 1301) - Testnet
* **Ink** (chainId: 57073) - Production

For complete network addresses and additional supported networks, see the [Contract Addresses](../../../resources/contract-addresses.md) documentation.

You can get free Base Sepolia ETH from the [Base Sepolia faucet](https://docs.base.org/tools/network-faucets) to test your applications.

## Error Handling

The SDK provides comprehensive error handling for V4-specific scenarios:

```typescript
try {
  const { createParams } = factory.buildConfig(config, addresses);
} catch (error) {
  if (error.message.includes('Invalid tick range')) {
    console.log('Tick range is invalid for the specified fee tier');
  } else if (error.message.includes('Hook mining failed')) {
    console.log('Could not find suitable hook address');
  } else {
    console.error('Unexpected error:', error);
  }
}
```

## Next Steps

* Explore the [Factory Reference](factory.md) for detailed API documentation
* Learn about [Doppler Lens Usage](lens.md) for advanced data querying
* Understand [Quoter Usage](quoter.md) for price calculations
* Review [Token Launch Examples](examples.md) for comprehensive deployment scenarios

## Support

For additional help and examples:

* Check the [Token Launch Examples](examples.md) for comprehensive deployment scenarios
* Review the [Implementation Guide](../../../how-it-works/implementation.md) for protocol details
* Join the community for discussions and support
