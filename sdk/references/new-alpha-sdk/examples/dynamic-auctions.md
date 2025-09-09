---
description: Create coins with Doppler's dynamic bonding curve, aka Doppler v4
---

# Dynamic auctions

```typescript
/**
 * Example: Create a Dynamic Auction with Uniswap V4 Migration
 *
 * This example demonstrates:
 * - Creating a gradual Dutch auction using Uniswap V4 hooks
 * - Configuring dynamic price movement with epochs
 * - Setting up V4 migration with fee streaming
 */

// UNCOMMENT IF RUNNING LOCALLY
// import { DopplerSDK, DynamicAuctionBuilder } from '@whetstone-research/doppler-sdk';

import { DopplerSDK } from '../src';

import {
  createPublicClient,
  createWalletClient,
  http,
  parseEther,
  formatEther,
} from 'viem';
import { base } from 'viem/chains';
import { privateKeyToAccount } from 'viem/accounts';

// Load environment variables
const privateKey = process.env.PRIVATE_KEY as `0x${string}`;
const rpcUrl = process.env.RPC_URL || 'https://mainnet.base.org' as string;

if (!privateKey) throw new Error('PRIVATE_KEY is not set');

async function main() {
  // 1. Set up clients
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

  // 2. Initialize SDK
  const sdk = new DopplerSDK({
    publicClient,
    walletClient,
    chainId: base.id,
  });

  // 3. Define dynamic auction parameters via builder
  const params = sdk.buildDynamicAuction()
    .tokenConfig({
      name: 'TEST DYNAMIC',
      symbol: 'TEST',
      tokenURI: 'https://example.com/dynamic-token.json',
    })
    .saleConfig({
      initialSupply: parseEther('10000000'), // 10M tokens
      numTokensToSell: parseEther('5000000'), // Sell 5M tokens
      numeraire: '0x4200000000000000000000000000000000000006', // WETH on Base
    })
    .poolConfig({ fee: 3000, tickSpacing: 60 })
    .auctionByTicks({
      durationDays: 7,
      epochLength: 3600,
      startTick: -92103,
      endTick: -69080,
      minProceeds: parseEther('100'),
      maxProceeds: parseEther('5000'),
    })
    .withMigration({
      type: 'uniswapV4',
      fee: 3000,
      tickSpacing: 60,
      streamableFees: {
        lockDuration: 365 * 24 * 60 * 60,
        beneficiaries: [
          { address: account.address, percentage: 10000 }, // 100%
          // Modify beneficiaries as needed
          // { address: '0xBeneficiary1...', percentage: 5000 }, // 50%
          // { address: '0xBeneficiary2...', percentage: 3000 }, // 30%
          // { address: '0xBeneficiary3...', percentage: 2000 }, // 20%
        ],
      },
    })
    .withGovernance({ type: 'default' })
    .withUserAddress(account.address)
    .build();

  console.log('Creating dynamic auction...');
  console.log('Token:', params.token.name, `(${params.token.symbol})`);
  console.log('Selling:', formatEther(params.sale.numTokensToSell), 'tokens');
  console.log('Duration:', params.auction.duration, 'days');
  console.log('Epochs:', params.auction.duration * 24, 'total (1 hour each)');
  console.log('Price will gradually increase from ~0.0001 to ~0.001 ETH');

  try {
    // 4. Create the dynamic auction
    const result = await sdk.factory.createDynamicAuction(params);

    console.log('\n✅ Dynamic auction created successfully!');
    console.log('Hook address:', result.hookAddress);
    console.log('Token address:', result.tokenAddress);
    console.log('Pool ID:', result.poolId);
    console.log('Transaction:', result.transactionHash);

    // 5. Get the auction instance
    const auction = await sdk.getDynamicAuction(result.hookAddress);

    // 6. Monitor the auction
    console.log('\nMonitoring dynamic auction...');

    const hookInfo = await auction.getHookInfo();
    console.log('Current epoch:', hookInfo.currentEpoch);
    console.log('Total proceeds:', formatEther(hookInfo.totalProceeds), 'ETH');
    console.log('Tokens sold:', formatEther(hookInfo.totalTokensSold));

    // 7. Get current price (as tick)
    const currentTick = await auction.getCurrentPrice();
    console.log('\nCurrent tick:', currentTick.toString());

    // 8. Check if auction ended early
    const hasEndedEarly = await auction.hasEndedEarly();
    if (hasEndedEarly) {
      console.log('\n🎯 Auction ended early - reached max proceeds!');
    } else {
      console.log('\nAuction is active. It will end when:');
      console.log('- All epochs complete (7 days)');
      console.log('- OR max proceeds reached (5000 ETH)');
      console.log('- OR insufficient demand detected');
    }

    // 9. Show graduation criteria
    const hasGraduated = await auction.hasGraduated();
    console.log('\nHas graduated:', hasGraduated);

    if (!hasGraduated) {
      console.log('After auction ends with sufficient proceeds:');
      console.log('- Liquidity migrates to Uniswap V4');
      console.log('- LP fees stream to beneficiaries over 1 year');
      console.log('- Token becomes freely tradeable');
    }
  } catch (error) {
    console.error('\n❌ Error creating dynamic auction:', error);
    process.exit(1);
  }

  console.log('\n✨ Example completed!');
}

main();
```
