---
description: Create coins with Doppler's Dutch auction bonding curve, aka Doppler v4
---

# Dynamic auctions

Dynamic auctions are Dutch auctions where the price starts high and descends over time through epochs until buyers purchase or the auction ends.

## Using market cap targets

`saleConfig()` and `poolConfig()` must be called before `withMarketCapRange()`.

```typescript
import { DopplerSDK } from '@whetstone-research/doppler-sdk';
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

  const params = sdk
    .buildDynamicAuction()
    .tokenConfig({
      name: 'My Token',
      symbol: 'MTK',
      tokenURI: 'https://example.com/token-metadata.json',
    })
    .saleConfig({
      initialSupply: parseEther('1000000000'),
      numTokensToSell: parseEther('500000000'),
      numeraire: '0x4200000000000000000000000000000000000006', // WETH on Base
    })
    .poolConfig({ fee: 3000, tickSpacing: 10 })
    .withMarketCapRange({
      marketCap: { start: 500_000, end: 50_000_000 }, // $500k to $50M
      numerairePrice: 3000, // ETH = $3000 USD
      minProceeds: parseEther('100'),
      maxProceeds: parseEther('5000'),
    })
    .withMigration({
      type: 'uniswapV4',
      fee: 3000,
      tickSpacing: 10,
      streamableFees: {
        lockDuration: 365 * 24 * 60 * 60,
        beneficiaries: [
          { beneficiary: account.address, shares: parseEther('0.95') },
          await sdk.getAirlockBeneficiary(),
        ],
      },
    })
    .withGovernance({ type: 'default' })
    .withUserAddress(account.address)
    .build();

  const result = await sdk.factory.createDynamicAuction(params);

  console.log('Hook:', result.hookAddress);
  console.log('Token:', result.tokenAddress);
  console.log('Pool ID:', result.poolId);
}

main();
```

---

## Using raw ticks

```typescript
const params = sdk.buildDynamicAuction()
  .tokenConfig({
    name: 'My Token',
    symbol: 'MTK',
    tokenURI: 'https://example.com/token-metadata.json',
  })
  .saleConfig({
    initialSupply: parseEther('1000000000'),
    numTokensToSell: parseEther('500000000'),
    numeraire: '0x4200000000000000000000000000000000000006',
  })
  .poolConfig({ fee: 3000, tickSpacing: 60 })
  .auctionByTicks({
    startTick: -92103,
    endTick: -69080,
    minProceeds: parseEther('100'),
    maxProceeds: parseEther('5000'),
    duration: 7 * 24 * 60 * 60,
    epochLength: 3600,
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
