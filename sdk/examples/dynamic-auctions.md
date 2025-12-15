---
description: Create coins with Doppler's dynamic bonding curve, aka Doppler v4
---

# Dynamic auctions

Dynamic auctions use Uniswap V4 hooks to create a gradual Dutch auction where the price descends over time through epochs. This allows for fair price discovery as the market determines the clearing price.

## Using market cap targets (recommended)

The `withMarketCapRange()` method is the recommended way to configure dynamic auctions. Instead of dealing with abstract tick values, you specify dollar-denominated market cap targets.

### Key requirements

1. `saleConfig()` must be called before `withMarketCapRange()`
2. `poolConfig()` must also be called before `withMarketCapRange()` (for tickSpacing)

### Example with market cap configuration

```typescript
/**
 * Example: Create a Dynamic Auction using Market Cap Range
 *
 * This example demonstrates:
 * - Configuring a V4 dynamic auction using dollar-denominated market cap targets
 * - The gradual Dutch auction mechanics with epochs
 * - Auto-detection of token ordering from numeraire address
 */

import { DopplerSDK, DAY_SECONDS } from '@whetstone-research/doppler-sdk';
import { parseEther, formatEther, createPublicClient, createWalletClient, http } from 'viem';
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

  // Build dynamic auction using market cap range instead of raw ticks
  // Note: both saleConfig() AND poolConfig() are required before withMarketCapRange()
  const params = sdk
    .buildDynamicAuction()
    .tokenConfig({
      name: 'My Token',
      symbol: 'MTK',
      tokenURI: 'https://example.com/token-metadata.json',
    })
    .saleConfig({
      initialSupply: parseEther('1000000000'), // 1 billion tokens
      numTokensToSell: parseEther('500000000'), // 500 million for sale
      numeraire: '0x4200000000000000000000000000000000000006', // WETH on Base
    })
    .poolConfig({ fee: 3000, tickSpacing: 10 }) // Required before withMarketCapRange!
    // Use market cap range - converts to ticks internally
    .withMarketCapRange({
      marketCap: { start: 500_000, end: 50_000_000 }, // $500k to $50M fully diluted
      numerairePrice: 3000, // ETH = $3000 USD
      minProceeds: parseEther('100'), // Min 100 ETH to graduate
      maxProceeds: parseEther('5000'), // Cap at 5000 ETH
      // Optional overrides (defaults shown):
      // duration: 7 * DAY_SECONDS,   // 7 day auction
      // epochLength: 3600,           // 1 hour epochs
      // gamma: <auto-calculated>,    // Tick decay per epoch
      // numPdSlugs: 5,               // Price discovery slugs
    })
    .withMigration({
      type: 'uniswapV4',
      fee: 3000,
      tickSpacing: 10,
      streamableFees: {
        lockDuration: 365 * 24 * 60 * 60, // 1 year
        beneficiaries: [
          { beneficiary: account.address, shares: parseEther('0.95') }, // 95%
          await sdk.getAirlockBeneficiary(), // 5% to protocol (default)
        ],
      },
    })
    .withGovernance({ type: 'default' })
    .withUserAddress(account.address)
    .build();

  console.log('Creating dynamic auction with market cap targets...');
  console.log('Token:', params.token.name, `(${params.token.symbol})`);
  console.log('Market cap range: $500,000 → $50,000,000 (at ETH = $3000)');
  console.log('Selling:', formatEther(params.sale.numTokensToSell), 'tokens');
  console.log('Computed ticks:', params.auction.startTick, '→', params.auction.endTick);
  console.log('Duration:', params.auction.duration / DAY_SECONDS, 'days');
  console.log('Epochs:', params.auction.duration / params.auction.epochLength, 'total');
  console.log('Proceeds range:', formatEther(params.auction.minProceeds), '→', formatEther(params.auction.maxProceeds), 'ETH');

  const result = await sdk.factory.createDynamicAuction(params);

  console.log('\n✅ Dynamic auction created successfully!');
  console.log('Hook address:', result.hookAddress);
  console.log('Token address:', result.tokenAddress);
  console.log('Pool ID:', result.poolId);
  console.log('Transaction:', result.transactionHash);

  // Monitor the auction
  const auction = await sdk.getDynamicAuction(result.hookAddress);

  // Wait for the transaction to be confirmed before reading contract state
  await auction.waitForDeployment(result.transactionHash as `0x${string}`);

  const hookInfo = await auction.getHookInfo();
  console.log('\nAuction Status:');
  console.log('Current epoch:', hookInfo.currentEpoch);
  console.log('Total proceeds:', formatEther(hookInfo.totalProceeds), 'ETH');
  console.log('Tokens sold:', formatEther(hookInfo.totalTokensSold));

  const hasEndedEarly = await auction.hasEndedEarly();
  if (hasEndedEarly) {
    console.log('\nAuction ended early - reached max proceeds!');
  } else {
    console.log('\nAuction is active. Will end when:');
    console.log('- All epochs complete (7 days)');
    console.log('- OR max proceeds reached (5000 ETH)');
  }

  const hasGraduated = await auction.hasGraduated();
  console.log('\nHas graduated:', hasGraduated);
}

main();
```

### Market cap configuration options

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `marketCap.start` | `number` | Required | Starting market cap in USD (auction ceiling) |
| `marketCap.end` | `number` | Required | Ending market cap in USD (auction floor) |
| `numerairePrice` | `number` | Required | Price of numeraire in USD (e.g., 3000 for ETH) |
| `minProceeds` | `bigint` | Required | Minimum ETH to graduate |
| `maxProceeds` | `bigint` | Required | Maximum ETH cap |
| `duration` | `number` | `604800` | Auction duration in seconds (7 days) |
| `epochLength` | `number` | `3600` | Epoch length in seconds (1 hour) |
| `gamma` | `number` | Auto-calculated | Tick decay per epoch |
| `numPdSlugs` | `number` | `5` | Price discovery slugs |
| `tokenDecimals` | `number` | `18` | Token decimals |
| `numeraireDecimals` | `number` | `18` | Numeraire decimals |

### How dynamic auctions work

1. **Dutch auction mechanics**: Price starts high and descends over time
2. **Epoch-based**: Each epoch (default: 1 hour), the price drops by `gamma` ticks
3. **Early exit**: If `maxProceeds` is reached, the auction ends early
4. **Graduation**: Once the auction ends with at least `minProceeds`, liquidity migrates

```
Auction starts at $50M market cap (high price)
        ↓
Price descends each epoch
        ↓
Buyers can purchase at any time
        ↓
Auction ends at $500k market cap (or when max proceeds reached)
        ↓
Liquidity migrates to destination pool
```

---

## Using raw ticks (advanced)

For advanced users who need precise control over tick values, you can use `auctionByTicks()` directly.

```typescript
const params = sdk.buildDynamicAuction()
  .tokenConfig({
    name: 'TEST DYNAMIC',
    symbol: 'TEST',
    tokenURI: 'https://example.com/dynamic-token.json',
  })
  .saleConfig({
    initialSupply: parseEther('10000000'),
    numTokensToSell: parseEther('5000000'),
    numeraire: '0x4200000000000000000000000000000000000006',
  })
  .poolConfig({ fee: 3000, tickSpacing: 60 })
  .auctionByTicks({
    startTick: -92103,
    endTick: -69080,
    minProceeds: parseEther('100'),
    maxProceeds: parseEther('5000'),
    duration: 7 * 24 * 60 * 60, // 7 days
    epochLength: 3600, // 1 hour
  })
  .withMigration({
    type: 'uniswapV4',
    fee: 3000,
    tickSpacing: 60,
    streamableFees: {
      lockDuration: 365 * 24 * 60 * 60,
      beneficiaries: [
        { beneficiary: account.address, shares: parseEther('1') },
      ],
    },
  })
  .withGovernance({ type: 'default' })
  .withUserAddress(account.address)
  .build();
```



---

## Comparison: Static vs Dynamic auctions

| Aspect | Static (V3) | Dynamic (V4) |
|--------|-------------|--------------|
| **Price movement** | Ascending (floor to ceiling) | Descending (ceiling to floor) |
| **Protocol** | Uniswap V3 | Uniswap V4 Hook |
| **Price discovery** | Fixed curve | Dutch auction with epochs |
| **Proceeds config** | Not configurable | `minProceeds`, `maxProceeds` required |
| **Duration** | N/A | Configurable with epochs |
| **Method order** | `saleConfig()` only | `saleConfig()` + `poolConfig()` |
