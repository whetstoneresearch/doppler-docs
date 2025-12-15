---
description: >-
  Create coins with Doppler Multicurve for richer liquidity distributions
---

# Multicurve

Multicurve pools use the Uniswap V4 initializer to seed pools with multiple overlapping curves in a single transaction. This enables sophisticated liquidity distributions that concentrate tokens at different market cap ranges.

## Using market cap ranges (recommended)

The `withCurves()` method is the recommended way to configure multicurve pools. No tick math required - just specify market cap ranges in USD for each curve.

### Key concepts

* **First curve's `marketCap.start`** = the launch price
* **Curves must be contiguous or overlapping** (no gaps allowed)
* **Shares must sum to exactly 1e18 (100%)**
* **Overlapping curves** provide extra liquidity at key price thresholds

### Example with market cap configuration

```typescript
/**
 * Example: Create a Multicurve Pool using Market Cap Ranges
 *
 * This example demonstrates:
 * - Configuring a V4 multicurve pool using dollar-denominated market cap ranges
 * - No tick math required - just specify market caps in USD
 * - Overlapping curves for extra liquidity at key thresholds
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

  // Build multicurve using market cap ranges - no tick math needed!
  const params = sdk
    .buildMulticurveAuction()
    .tokenConfig({
      name: 'My Token',
      symbol: 'MTK',
      tokenURI: 'https://example.com/token-metadata.json',
    })
    .saleConfig({
      initialSupply: parseEther('1000000000'), // 1 billion tokens
      numTokensToSell: parseEther('900000000'), // 900 million for sale (90%)
      numeraire: '0x4200000000000000000000000000000000000006', // WETH on Base
    })
    // Use market cap ranges - the first curve's start is the launch price
    .withCurves({
      numerairePrice: 3000, // ETH = $3000 USD
      curves: [
        // Curve 1: Launch curve - starts at $500k market cap
        {
          marketCap: { start: 500_000, end: 1_500_000 }, // $500k - $1.5M
          numPositions: 10,
          shares: parseEther('0.3'), // 30% of tokens
        },
        // Curve 2: Overlaps with first curve at $1M for extra liquidity
        {
          marketCap: { start: 1_000_000, end: 5_000_000 }, // $1M - $5M
          numPositions: 15,
          shares: parseEther('0.4'), // 40% of tokens
        },
        // Curve 3: Upper range with overlap at $4-5M (moon bag)
        {
          marketCap: { start: 4_000_000, end: 50_000_000 }, // $4M - $50M
          numPositions: 10,
          shares: parseEther('0.3'), // 30% of tokens
        },
      ],
      // Optional overrides:
      // fee: 500,             // 0.05% (default for multicurve)
      // tickSpacing: 10,      // Derived from fee if not provided
      // beneficiaries: [...], // Optional fee streaming recipients
    })
    .withVesting({
      duration: BigInt(365 * 24 * 60 * 60), // 1 year vesting
      cliffDuration: 0,
    })
    .withGovernance({ type: 'default' })
    .withMigration({ type: 'uniswapV2' }) // V2 migration for simplicity
    .withUserAddress(account.address)
    .build();

  console.log('Creating multicurve pool with market cap targets...');
  console.log('Token:', params.token.name, `(${params.token.symbol})`);
  console.log('Launch market cap: $500,000 (at ETH = $3000)');
  console.log('Curves:', params.pool.curves.length, 'configured');

  // Log curve details
  params.pool.curves.forEach((curve, i) => {
    console.log(`  Curve ${i + 1}: ticks ${curve.tickLower} -> ${curve.tickUpper}, ${curve.numPositions} positions`);
  });

  // Simulate to preview addresses and get executable
  const simulation = await sdk.factory.simulateCreateMulticurve(params);
  console.log('\nPredicted addresses:');
  console.log('Token:', simulation.asset);
  console.log('Pool:', simulation.pool);
  console.log('Gas estimate:', simulation.gasEstimate);

  // Execute with guaranteed same addresses (uses same salt from simulation)
  const result = await simulation.execute();

  console.log('\n✅ Multicurve pool created successfully!');
  console.log('Token address:', result.tokenAddress);
  console.log('Pool address:', result.poolAddress);
  console.log('Transaction:', result.transactionHash);

  // Get the pool instance for monitoring
  const poolInstance = await sdk.getMulticurvePool(result.poolAddress);
  const state = await poolInstance.getState();

  console.log('\nPool Info:');
  console.log('Fee:', state.poolKey.fee);
  console.log('Tick spacing:', state.poolKey.tickSpacing);
  console.log('Status:', state.status);
}

main();
```

### How curve overlapping works

Overlapping curves provide extra liquidity at key market cap thresholds:

```
$500k         $1M          $1.5M        $4M         $5M         $50M
  |-------------|-------------|                                    Curve 1 (30%)
                |---------------------------|                       Curve 2 (40%)
                                     |----------------------------| Curve 3 (30%)

                ↑               ↑
           Overlap 1        Overlap 2
         ($1M-$1.5M)       ($4M-$5M)
       Extra liquidity   Extra liquidity
```

### Market cap curve configuration options

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `numerairePrice` | `number` | Required | Price of numeraire in USD (e.g., 3000 for ETH) |
| `curves` | `array` | Required | Array of curve configurations |
| `curves[].marketCap.start` | `number` | Required | Start market cap in USD |
| `curves[].marketCap.end` | `number` | Required | End market cap in USD |
| `curves[].numPositions` | `number` | Required | Number of LP positions |
| `curves[].shares` | `bigint` | Required | Share in WAD (e.g., `parseEther('0.3')` = 30%) |
| `fee` | `number` | `500` | Fee tier (0.05% default) |
| `tickSpacing` | `number` | Auto | Derived from fee if not provided |
| `beneficiaries` | `array` | `undefined` | Optional fee streaming recipients |

### Validation rules

The SDK validates multicurve configurations:

* **Shares must sum to 100%**: Total of all curve shares must equal `1e18`
* **No gaps allowed**: Curves must be contiguous or overlapping
* **Market cap ordering**: `start < end` for each curve
* **Positive values**: All market caps and positions must be > 0

---

## Three-tier distribution strategy

A common pattern is to use three curves for different investor profiles:

```typescript
.withCurves({
  numerairePrice: 3000,
  curves: [
    // Launch curve: Early buyers get concentrated liquidity
    { marketCap: { start: 500_000, end: 1_500_000 }, numPositions: 10, shares: parseEther('0.3') },
    
    // Mid-range: Provides depth as price rises
    { marketCap: { start: 1_000_000, end: 5_000_000 }, numPositions: 15, shares: parseEther('0.4') },
    
    // Moon bag: Liquidity for high market cap targets
    { marketCap: { start: 4_000_000, end: 50_000_000 }, numPositions: 10, shares: parseEther('0.3') },
  ],
})
```

### Common market cap ranges

| Launch Size | Start | End | Description |
|-------------|-------|-----|-------------|
| Small | $10k | $1M | Micro cap, high risk/reward |
| Medium | $100k | $10M | Standard token launch |
| Large | $1M | $100M | Established project |

---

## Using raw ticks (advanced)

For advanced users who need precise control over tick values, you can use `poolConfig()` directly.

```typescript
import { DopplerSDK, WAD } from '@whetstone-research/doppler-sdk';
import { parseEther } from 'viem';

const params = sdk
  .buildMulticurveAuction()
  .tokenConfig({ name: 'TEST', symbol: 'TEST', tokenURI: 'ipfs://token.json' })
  .saleConfig({
    initialSupply: 1_000_000n * WAD,
    numTokensToSell: 900_000n * WAD,
    numeraire: '0x4200000000000000000000000000000000000006',
  })
  .poolConfig({
    fee: 3000,
    tickSpacing: 60,
    curves: [
      { tickLower: -120000, tickUpper: -90000, numPositions: 8, shares: parseEther('0.4') },
      { tickLower: -90000, tickUpper: -70000, numPositions: 8, shares: parseEther('0.6') },
    ],
  })
  .withGovernance({ type: 'default' })
  .withMigration({ type: 'uniswapV2' })
  .withUserAddress(account.address)
  .build();
```

---

## Comparison: Auction types

| Aspect | Static (V3) | Dynamic (V4) | Multicurve (V4) |
|--------|-------------|--------------|-----------------|
| **Method** | `withMarketCapRange()` | `withMarketCapRange()` | `withCurves()` |
| **Protocol** | Uniswap V3 | Uniswap V4 Hook | Uniswap V4 Initializer |
| **Price movement** | Ascending curve | Descending Dutch auction | Multiple overlapping curves |
| **Multiple curves** | No | No | Yes |
| **Proceeds config** | Not configurable | `minProceeds`, `maxProceeds` | N/A |
| **Default fee** | 10000 (1%) | User-specified | 500 (0.05%) |
