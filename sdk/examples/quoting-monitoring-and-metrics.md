---
description: >-
  Examples for getting swap quotes, monitoring auction progress, and fetching
  token details
---

# Quoting, monitoring, and metrics

### Price quotes

```typescript
/**
 * Example: Price Quoter
 * 
 * This example demonstrates:
 * - Getting price quotes across Uniswap V2, V3, and V4
 * - Comparing quotes to find best prices
 * - Handling different swap types (exact input/output)
 */

// UNCOMMENT IF RUNNING LOCALLY
// import { DopplerSDK } from '@whetstone-research/doppler-sdk';
import { DopplerSDK } from '../src';

import { createPublicClient, http, parseEther, formatEther, type Address } from 'viem'
import { base } from 'viem/chains'

const token = process.env.TOKEN as `0x${string}`;
const rpcUrl = process.env.RPC_URL || 'https://mainnet.base.org' as string;

if (!token) throw new Error('TOKEN is not set');

// Example token addresses (replace with actual addresses)
const weth = '0x4200000000000000000000000000000000000006' as Address // WETH on Base
const usdc = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913' as Address // USDC on Base

async function main() {
  // Initialize SDK
  const publicClient = createPublicClient({
    chain: base,
    transport: http(rpcUrl)
  })

  const sdk = new DopplerSDK({
    publicClient,
    chainId: base.id
  })

  const quoter = sdk.quoter

  console.log('💱 Price Quoter Example')
  console.log('=====================')

  // Example 1: Quote exact input on V3
  console.log('\n📊 Example 1: Swap 1 ETH for USDC on V3')
  try {
    const v3Quote = await quoter.quoteExactInputV3({
      tokenIn: weth,
      tokenOut: usdc,
      amountIn: parseEther('1'),
      fee: 3000 // 0.3% fee tier
    })
    
    console.log('- Amount out:', formatEther(v3Quote.amountOut), 'USDC')
    console.log('- Price impact (ticks crossed):', v3Quote.initializedTicksCrossed)
    console.log('- Gas estimate:', v3Quote.gasEstimate.toString())
    console.log('- Final sqrtPriceX96:', v3Quote.sqrtPriceX96After.toString())
  } catch (error) {
    console.error('V3 quote failed:', error.message)
  }

  // Example 2: Quote exact output on V3
  console.log('\n📊 Example 2: Get exactly 2000 USDC, pay in ETH on V3')
  try {
    const v3QuoteOut = await quoter.quoteExactOutputV3({
      tokenIn: weth,
      tokenOut: usdc,
      amountOut: parseEther('2000'), // Want exactly 2000 USDC
      fee: 3000
    })
    
    console.log('- Amount in required:', formatEther(v3QuoteOut.amountIn), 'ETH')
    console.log('- Price impact (ticks crossed):', v3QuoteOut.initializedTicksCrossed)
    console.log('- Gas estimate:', v3QuoteOut.gasEstimate.toString())
  } catch (error) {
    console.error('V3 exact output quote failed:', error.message)
  }

  // Example 3: Quote on V2 (if available)
  console.log('\n📊 Example 3: Swap 1 ETH for USDC on V2')
  try {
    const v2Quote = await quoter.quoteExactInputV2({
      amountIn: parseEther('1'),
      path: [weth, usdc]
    })
    
    console.log('- Amount out:', formatEther(v2Quote[1]), 'USDC')
    console.log('- Simple constant product AMM pricing')
  } catch (error) {
    console.error('V2 quote failed:', error.message)
  }

  // Example 4: Multi-hop V2 quote
  console.log('\n📊 Example 4: Multi-hop swap ETH -> USDC -> TOKEN on V2')
  try {
    const multiHopQuote = await quoter.quoteExactInputV2({
      amountIn: parseEther('1'),
      path: [weth, usdc, token] // ETH -> USDC -> TOKEN
    })
    
    console.log('Hop results:')
    console.log('- Start:', formatEther(multiHopQuote[0]), 'ETH')
    console.log('- After hop 1:', formatEther(multiHopQuote[1]), 'USDC')
    console.log('- Final:', formatEther(multiHopQuote[2]), 'TOKEN')
  } catch (error) {
    console.error('Multi-hop quote failed:', error.message)
  }

  // Example 5: V4 quote (for graduated dynamic auctions)
  console.log('\n📊 Example 5: Swap on V4 pool')
  try {
    const v4PoolKey = {
      currency0: weth,
      currency1: token,
      fee: 3000,
      tickSpacing: 60,
      hooks: '0x0000000000000000000000000000000000000000' as Address // No hook for graduated pool
    }
    
    const v4Quote = await quoter.quoteExactInputV4({
      poolKey: v4PoolKey,
      zeroForOne: true, // Swapping currency0 (WETH) for currency1 (TOKEN)
      exactAmount: parseEther('1')
    })
    
    console.log('- Amount out:', formatEther(v4Quote.amountOut), 'TOKEN')
    console.log('- Gas estimate:', v4Quote.gasEstimate.toString())
  } catch (error) {
    console.error('V4 quote failed:', error.message)
  }

  // Example 6: Compare quotes across versions
  console.log('\n🔄 Comparing quotes for 1 ETH -> USDC:')
  const results: { version: string; amountOut: bigint; gas: bigint }[] = []
  
  // Try V2
  try {
    const v2 = await quoter.quoteExactInputV2({
      amountIn: parseEther('1'),
      path: [weth, usdc]
    })
    results.push({ 
      version: 'V2', 
      amountOut: v2[1], 
      gas: BigInt(100000) // Approximate
    })
  } catch {}
  
  // Try V3
  try {
    const v3 = await quoter.quoteExactInputV3({
      tokenIn: weth,
      tokenOut: usdc,
      amountIn: parseEther('1'),
      fee: 3000
    })
    results.push({ 
      version: 'V3', 
      amountOut: v3.amountOut, 
      gas: v3.gasEstimate 
    })
  } catch {}
  
  // Sort by best output
  results.sort((a, b) => Number(b.amountOut - a.amountOut))
  
  console.log('\nBest quotes (sorted by output):')
  results.forEach((result, i) => {
    console.log(`${i + 1}. ${result.version}: ${formatEther(result.amountOut)} USDC (gas: ${result.gas})`)
  })
  
  if (results.length > 0) {
    console.log(`\n✅ Best option: ${results[0].version} with ${formatEther(results[0].amountOut)} USDC`)
  }

  console.log('\n✨ Example completed!')
}

main()
```

### Auction monitoring

```typescript
/**
 * Example: Monitor Existing Auctions
 *
 * This example demonstrates:
 * - Monitoring static and dynamic auctions
 * - Checking graduation status
 * - Tracking key metrics and progress
 */

// UNCOMMENT IF RUNNING LOCALLY
// import { DopplerSDK } from '@whetstone-research/doppler-sdk';

import { DopplerSDK } from '../src';
import {
  http,
  formatEther,
  type Address,
  createPublicClient,
} from 'viem';
import { base } from 'viem/chains';

// Example addresses - replace with your actual auction addresses
const staticPoolAddress = process.env.STATIC_POOL_ADDRESS as `0x${string}`;
const dynamicHookAddress = process.env.DYNAMIC_HOOK_ADDRESS as `0x${string}`;
const rpcUrl = process.env.RPC_URL || ('https://mainnet.base.org' as string);

async function monitorStaticAuction(sdk: DopplerSDK, poolAddress: Address) {
  console.log('\n📊 Monitoring Static Auction...');
  console.log('Pool address:', poolAddress);

  try {
    const auction = await sdk.getStaticAuction(poolAddress);

    // Get pool information
    const poolInfo = await auction.getPoolInfo();
    console.log('\nPool Information:');
    console.log('- Token:', poolInfo.tokenAddress);
    console.log('- Numeraire:', poolInfo.numeraireAddress);
    console.log('- Fee tier:', poolInfo.fee / 10000, '%');
    console.log('- Liquidity:', formatEther(poolInfo.liquidity));
    console.log('- SqrtPriceX96:', poolInfo.sqrtPriceX96.toString());

    // Get current price
    const currentPrice = await auction.getCurrentPrice();
    console.log('\nCurrent tick price:', currentPrice.toString());

    // Check graduation status
    const hasGraduated = await auction.hasGraduated();
    console.log(
      '\nGraduation status:',
      hasGraduated ? '✅ Graduated' : '⏳ Active'
    );

    // Get token address for further info
    const tokenAddress = await auction.getTokenAddress();
    console.log('Token contract:', tokenAddress);
  } catch (error) {
    console.error('Error monitoring static auction:', error);
  }
}

async function monitorDynamicAuction(sdk: DopplerSDK, hookAddress: Address) {
  console.log('\n📊 Monitoring Dynamic Auction...');
  console.log('Hook address:', hookAddress);

  try {
    const auction = await sdk.getDynamicAuction(hookAddress);

    // Get comprehensive hook information
    const hookInfo = await auction.getHookInfo();
    console.log('\nHook Information:');
    console.log('- Token:', hookInfo.tokenAddress);
    console.log('- Numeraire:', hookInfo.numeraireAddress);
    console.log('- Pool ID:', hookInfo.poolId);
    console.log('- Current epoch:', hookInfo.currentEpoch);

    console.log('\nSale Progress:');
    console.log(
      '- Total proceeds:',
      formatEther(hookInfo.totalProceeds),
      'ETH'
    );
    console.log('- Tokens sold:', formatEther(hookInfo.totalTokensSold));
    console.log(
      '- Min proceeds:',
      formatEther(hookInfo.minimumProceeds),
      'ETH'
    );
    console.log(
      '- Max proceeds:',
      formatEther(hookInfo.maximumProceeds),
      'ETH'
    );

    console.log('\nAuction Status:');
    console.log('- Early exit:', hookInfo.earlyExit ? '✅ Yes' : '❌ No');
    console.log(
      '- Insufficient proceeds:',
      hookInfo.insufficientProceeds ? '⚠️ Yes' : '✅ No'
    );

    // Calculate time remaining
    const now = BigInt(Math.floor(Date.now() / 1000));
    if (now < hookInfo.endingTime) {
      const remaining = Number(hookInfo.endingTime - now);
      const hours = Math.floor(remaining / 3600);
      const minutes = Math.floor((remaining % 3600) / 60);
      console.log('- Time remaining:', `${hours}h ${minutes}m`);
    } else {
      console.log('- Time remaining: Auction ended');
    }

    // Get current price tick
    const currentTick = await auction.getCurrentPrice();
    console.log('\nCurrent tick:', currentTick.toString());

    // Check if ended early
    const hasEndedEarly = await auction.hasEndedEarly();
    if (hasEndedEarly) {
      console.log('\n🎯 Auction ended early due to reaching max proceeds');
    }

    // Check graduation
    const hasGraduated = await auction.hasGraduated();
    console.log(
      'Graduation status:',
      hasGraduated ? '✅ Graduated' : '⏳ Active'
    );
  } catch (error) {
    console.error('Error monitoring dynamic auction:', error);
  }
}

async function main() {
  // Initialize SDK in read-only mode
  const publicClient = createPublicClient({
    chain: base,
    transport: http(rpcUrl),
  });

  const sdk = new DopplerSDK({
    publicClient,
    chainId: base.id,
  });

  console.log('🔍 Doppler Auction Monitor');
  console.log('=======================');

  // Monitor static auction if address provided
  if (staticPoolAddress) {
    await monitorStaticAuction(sdk, staticPoolAddress);
  } else {
    console.log('\n⚠️  No static auction address provided');
  }

  // Monitor dynamic auction if address provided
  if (dynamicHookAddress) {
    await monitorDynamicAuction(sdk, dynamicHookAddress);
  } else {
    console.log('\n⚠️  No dynamic auction address provided');
  }

  console.log('\n✨ Monitoring complete!');
}

main()
```

### Token interactions

```typescript
/**
 * Example: Token Interaction
 *
 * This example demonstrates:
 * - Interacting with DERC20 tokens launched via Doppler
 * - Checking balances and vesting data
 * - Approving spending and releasing vested tokens
 */

// UNCOMMENT IF RUNNING LOCALLY
// import { Derc20, Eth } from '@whetstone-research/doppler-sdk';

import { Derc20, Eth } from '../src';
import {
  createPublicClient,
  createWalletClient,
  http,
  parseEther,
  formatEther,
} from 'viem';
import { base } from 'viem/chains';
import { privateKeyToAccount } from 'viem/accounts';

// Configuration
const spender = process.env.SPENDER as `0x${string}`;
const privateKey = process.env.PRIVATE_KEY as `0x${string}`;
const rpcUrl = process.env.RPC_URL || 'https://mainnet.base.org' as string;
const tokenAddress = process.env.TOKEN_ADDRESS as `0x${string}`;

if (!privateKey) throw new Error('PRIVATE_KEY is not set');
if (!tokenAddress) throw new Error('TOKEN_ADDRESS is not set');
if (!spender) throw new Error('SPENDER is not set');

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

  console.log('💰 Token Interaction Example');
  console.log('===========================');
  console.log('Account:', account.address);
  console.log('Token:', tokenAddress);

  // 2. Create token instance
  const token = new Derc20(publicClient, walletClient, tokenAddress);

  try {
    // 3. Get token information
    console.log('\n📋 Token Information:');
    const [name, symbol, decimals, totalSupply] = await Promise.all([
      token.getName(),
      token.getSymbol(),
      token.getDecimals(),
      token.getTotalSupply(),
    ]);

    console.log('- Name:', name);
    console.log('- Symbol:', symbol);
    console.log('- Decimals:', decimals);
    console.log('- Total Supply:', formatEther(totalSupply), symbol);

    // 4. Check balances
    console.log('\n💸 Balances:');
    const balance = await token.getBalanceOf(account.address);
    console.log('- Your balance:', formatEther(balance), symbol);

    // Also check ETH balance
    const eth = new Eth(publicClient);
    const ethBalance = await eth.getBalanceOf(account.address);
    console.log('- ETH balance:', formatEther(ethBalance), 'ETH');

    // 5. Check vesting information
    console.log('\n⏰ Vesting Information:');
    const [vestingDuration, vestingStart, vestedTotal] = await Promise.all([
      token.getVestingDuration(),
      token.getVestingStart(),
      token.getVestedTotalAmount(),
    ]);

    if (vestingDuration > 0n) {
      const vestingEndTime = vestingStart + vestingDuration;
      const now = BigInt(Math.floor(Date.now() / 1000));
      const isVestingActive = now < vestingEndTime;

      console.log(
        '- Vesting duration:',
        Number(vestingDuration) / 86400,
        'days'
      );
      console.log(
        '- Vesting start:',
        new Date(Number(vestingStart) * 1000).toLocaleString()
      );
      console.log('- Total vested amount:', formatEther(vestedTotal), symbol);
      console.log('- Vesting active:', isVestingActive);

      // Check user's vesting data
      const vestingData = await token.getVestingData(account.address);
      console.log('\n📊 Your Vesting Data:');
      console.log(
        '- Total vested:',
        formatEther(vestingData.totalAmount),
        symbol
      );
      console.log(
        '- Already released:',
        formatEther(vestingData.releasedAmount),
        symbol
      );

      // Calculate available to release
      const available = await token.getAvailableVestedAmount(account.address);
      console.log('- Available to release:', formatEther(available), symbol);

      // Release vested tokens if available
      if (available > 0n) {
        console.log('\n🎯 Releasing vested tokens...');
        try {
          const txHash = await token.release(available);
          console.log('✅ Tokens released! Transaction:', txHash);
        } catch (error) {
          console.error('❌ Failed to release tokens:', error);
        }
      }
    } else {
      console.log('- No vesting configured for this token');
    }

    // 6. Token approvals
    console.log('\n🔓 Token Approvals:');
    const currentAllowance = await token.getAllowance(account.address, spender);
    console.log('- Current allowance:', formatEther(currentAllowance), symbol);

    // Approve spending if needed
    const approvalAmount = parseEther('100');
    if (currentAllowance < approvalAmount) {
      console.log(
        `\n📝 Approving ${formatEther(approvalAmount)} ${symbol} for spender...`
      );
      try {
        const txHash = await token.approve(spender, approvalAmount);
        console.log('✅ Approval successful! Transaction:', txHash);
      } catch (error) {
        console.error('❌ Approval failed:', error);
      }
    }

    // 7. Additional token info
    console.log('\n🔍 Additional Information:');
    const [tokenURI, pool, isPoolUnlocked, yearlyMintRate] = await Promise.all([
      token.getTokenURI(),
      token.getPool(),
      token.getIsPoolUnlocked(),
      token.getYearlyMintRate(),
    ]);

    console.log('- Token URI:', tokenURI);
    console.log('- Pool address:', pool);
    console.log('- Pool unlocked:', isPoolUnlocked);
    console.log(
      '- Yearly mint rate:',
      formatEther(yearlyMintRate),
      symbol,
      'per year'
    );
  } catch (error) {
    console.error('\n❌ Error:', error);
    process.exit(1);
  }

  console.log('\n✨ Example completed!');
}

main();
```
