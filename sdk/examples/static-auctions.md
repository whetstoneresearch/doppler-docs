---
description: Create coins with Doppler's static bonding curve, aka Doppler v3
---

# Static auctions

Static auctions use Uniswap V3's concentrated liquidity to create a fixed price range bonding curve. The price ascends from a floor to a ceiling as tokens are purchased.

## Using market cap targets (recommended)

The `withMarketCapRange()` method is the recommended way to configure static auctions. Instead of dealing with abstract tick values, you specify dollar-denominated market cap targets.

### Benefits

* **Intuitive**: Think in dollars, not ticks
* **Auto-detection**: Token ordering (`tokenIsToken1`) inferred from numeraire address
* **Validation**: Built-in checks for reasonable market cap ranges

### Example with Uniswap V2 migration

```typescript
/**
 * Example: Create a Static Auction using Market Cap Range
 *
 * This example demonstrates:
 * - Configuring a V3 static auction using dollar-denominated market cap targets
 * - Auto-detection of token ordering from numeraire address
 * - Simplified API that converts market cap to ticks internally
 */

import { DopplerSDK } from '@whetstone-research/doppler-sdk';
import { parseEther, createPublicClient, createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';

const privateKey = process.env.PRIVATE_KEY as `0x${string}`;
const rpcUrl = process.env.RPC_URL ?? 'https://mainnet.base.org';

if (!privateKey) throw new Error('PRIVATE_KEY must be set');

async function main() {
  const account = privateKeyToAccount(privateKey);

  const publicClient = createPublicClient({
    chain: base,
    transport: http(rpcUrl),
  });

  const walletClient = createWalletClient({
    chain: base,
    transport: http(rpcUrl),
    account,
  });

  const sdk = new DopplerSDK({
    publicClient,
    walletClient,
    chainId: base.id,
  });

  // Build static auction using market cap range instead of raw ticks
  const params = sdk
    .buildStaticAuction()
    .tokenConfig({
      name: 'My Token',
      symbol: 'MTK',
      tokenURI: 'https://example.com/token-metadata.json',
    })
    .saleConfig({
      initialSupply: parseEther('1000000000'), // 1 billion tokens
      numTokensToSell: parseEther('900000000'), // 900 million for sale
      numeraire: '0x4200000000000000000000000000000000000006', // WETH on Base
    })
    // Use market cap range - much more intuitive than raw ticks!
    .withMarketCapRange({
      marketCap: { start: 100_000, end: 10_000_000 }, // $100k to $10M fully diluted
      numerairePrice: 3000, // ETH = $3000 USD
      // Optional overrides (defaults shown):
      // fee: 10000,            // 1% fee tier (default for V3)
      // numPositions: 15,      // Number of LP positions
      // maxShareToBeSold: parseEther('0.35'), // 35% max per position
    })
    .withVesting({
      duration: BigInt(365 * 24 * 60 * 60), // 1 year vesting
      cliffDuration: 0,
    })
    .withGovernance({ type: 'default' })
    .withMigration({ type: 'uniswapV2' })
    .withUserAddress(account.address)
    .build();

  console.log('Creating static auction with market cap targets...');
  console.log('Token:', params.token.name, `(${params.token.symbol})`);
  console.log('Market cap range: $100,000 → $10,000,000 (at ETH = $3000)');
  console.log('Computed ticks:', params.pool.startTick, '→', params.pool.endTick);

  const result = await sdk.factory.createStaticAuction(params);

  console.log('\n✅ Static auction created successfully!');
  console.log('Pool address:', result.poolAddress);
  console.log('Token address:', result.tokenAddress);
  console.log('Transaction:', result.transactionHash);

  // Monitor the auction
  const auction = await sdk.getStaticAuction(result.poolAddress);
  const hasGraduated = await auction.hasGraduated();

  if (hasGraduated) {
    console.log('\nAuction has graduated - ready for migration!');
  } else {
    console.log('\nAuction is active. Will graduate when sufficient tokens are sold.');
  }
}

main();
```

### Market cap configuration options

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `marketCap.start` | `number` | Required | Starting market cap in USD (launch price) |
| `marketCap.end` | `number` | Required | Ending market cap in USD |
| `numerairePrice` | `number` | Required | Price of numeraire in USD (e.g., 3000 for ETH) |
| `fee` | `number` | `10000` | Fee tier in basis points (1% default) |
| `numPositions` | `number` | `15` | Number of LP positions |
| `maxShareToBeSold` | `bigint` | `0.35e18` | Maximum share per position (35%) |
| `tokenDecimals` | `number` | `18` | Token decimals |
| `numeraireDecimals` | `number` | `18` | Numeraire decimals |

### Example with Uniswap V4 migration

```typescript
import { DopplerSDK, getAirlockOwner } from '@whetstone-research/doppler-sdk';
import { parseEther, createPublicClient, createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';

const privateKey = process.env.PRIVATE_KEY as `0x${string}`;
const rpcUrl = process.env.RPC_URL ?? 'https://mainnet.base.org';

async function main() {
  const account = privateKeyToAccount(privateKey);

  const publicClient = createPublicClient({
    chain: base,
    transport: http(rpcUrl),
  });

  const walletClient = createWalletClient({
    chain: base,
    transport: http(rpcUrl),
    account,
  });

  const sdk = new DopplerSDK({
    publicClient,
    walletClient,
    chainId: base.id,
  });

  const airlockOwner = await getAirlockOwner(publicClient);

  const params = sdk
    .buildStaticAuction()
    .tokenConfig({
      name: 'My Token',
      symbol: 'MTK',
      tokenURI: 'https://example.com/token-metadata.json',
    })
    .saleConfig({
      initialSupply: parseEther('1000000000'),
      numTokensToSell: parseEther('900000000'),
      numeraire: '0x4200000000000000000000000000000000000006',
    })
    .withMarketCapRange({
      marketCap: { start: 100_000, end: 10_000_000 },
      numerairePrice: 3000,
    })
    .withVesting({
      duration: BigInt(365 * 24 * 60 * 60),
      cliffDuration: 0,
    })
    .withGovernance({ type: 'default' })
    .withMigration({
      type: 'uniswapV4',
      fee: 3000,
      tickSpacing: 60,
      streamableFees: {
        lockDuration: 365 * 24 * 60 * 60,
        beneficiaries: [
          { beneficiary: account.address, shares: parseEther('0.95') },
          { beneficiary: airlockOwner, shares: parseEther('0.05') },
        ],
      },
    })
    .withUserAddress(account.address)
    .build();

  const result = await sdk.factory.createStaticAuction(params);

  console.log('Pool created:', result.poolAddress);
  console.log('Token created:', result.tokenAddress);

  const auction = await sdk.getStaticAuction(result.poolAddress);
  const hasGraduated = await auction.hasGraduated();

  if (hasGraduated) {
    console.log('Auction is ready for V4 migration!');
  }
}

main();
```

---

## Using raw ticks (advanced)

For advanced users who need precise control over tick values, you can use `poolByTicks()` directly.

```typescript
const params = sdk.buildStaticAuction()
  .tokenConfig({
    name: "TEST",
    symbol: "TEST",
    tokenURI: "https://example.com/token-metadata.json",
  })
  .saleConfig({
    initialSupply: parseEther("1000000"),
    numTokensToSell: parseEther("500000"),
    numeraire: "0x4200000000000000000000000000000000000006",
  })
  .poolByTicks({ startTick: 175000, endTick: 225000, fee: 10000 })
  .withMigration({ type: "uniswapV2" })
  .withUserAddress(account.address)
  .withGovernance({ type: 'default' })
  .build();
```



### With proceeds migration to Uniswap v4&#x20;

```typescript
import { DopplerSDK, getAirlockOwner } from '../src';
import { parseEther, createPublicClient, createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';

const privateKey = process.env.PRIVATE_KEY as `0x${string}`;
const rpcUrl = process.env.RPC_URL ?? "https://mainnet.base.org";
const account = privateKeyToAccount(privateKey);

if (!privateKey) throw new Error('PRIVATE_KEY must be set');

// Example: Creating a static auction that migrates to Uniswap V4
async function createStaticAuctionExample() {
  // Create viem clients
  const publicClient = createPublicClient({
    chain: base,
    transport: http(rpcUrl),
  });

  const walletClient = createWalletClient({
    chain: base,
    transport: http(rpcUrl),
    account: account,
  });

  if (!publicClient || !walletClient) {
    throw new Error('Failed to create viem clients');
  }

  // Initialize the SDK
  const sdk = new DopplerSDK({
    publicClient,
    walletClient,
    chainId: 8453, // Base mainnet
  });

  const airlockOwner = await getAirlockOwner(publicClient);

  // Configure the static auction with the builder
  const params = sdk
    .buildStaticAuction()
    .tokenConfig({
      name: 'My Token',
      symbol: 'MTK',
      tokenURI: 'https://example.com/token-metadata.json',
    })
    .saleConfig({
      initialSupply: parseEther('1000000000'), // 1 billion tokens
      numTokensToSell: parseEther('900000000'), // 900 million for sale
      numeraire: '0x4200000000000000000000000000000000000006', // WETH on Base
    })
    .poolByTicks({ startTick: 175000, endTick: 225000, fee: 3000 })
    .withVesting({ duration: BigInt(365 * 24 * 60 * 60), cliffDuration: 0 })
    .withMigration({
      type: 'uniswapV4',
      fee: 3000,
      tickSpacing: 60,
      streamableFees: {
        lockDuration: 365 * 24 * 60 * 60, // 1 year
        beneficiaries: [
          { address: account.address, percentage: 9500 }, // 95%
          { address: airlockOwner, percentage: 500 }, // 5%
        ],
      },
    })
    .withUserAddress(account.address)
    .withGovernance({ type: "default" })
    .build();

  // Create the static auction
  const result = await sdk.factory.createStaticAuction(params);

  console.log('Pool created:', result.poolAddress);
  console.log('Token created:', result.tokenAddress);
  console.log('Transaction:', result.transactionHash);

  // Later, interact with the auction
  const auction = await sdk.getStaticAuction(result.poolAddress);
  const hasGraduated = await auction.hasGraduated();

  if (hasGraduated) {
    console.log('Auction is ready for migration!');
  }
}

const main = async () => {
  await createStaticAuctionExample();
};

main();
```
