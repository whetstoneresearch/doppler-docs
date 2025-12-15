---
description: Create coins with Doppler's static bonding curve, aka Doppler v3
---

# Static auctions

## Using market cap targets

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
    .buildStaticAuction()
    .tokenConfig({
      name: 'My Token',
      symbol: 'MTK',
      tokenURI: 'https://example.com/token-metadata.json',
    })
    .saleConfig({
      initialSupply: parseEther('1000000000'),
      numTokensToSell: parseEther('900000000'),
      numeraire: '0x4200000000000000000000000000000000000006', // WETH on Base
    })
    .withMarketCapRange({
      marketCap: { start: 100_000, end: 10_000_000 }, // $100k to $10M
      numerairePrice: 3000, // ETH = $3000 USD
    })
    .withVesting({
      duration: BigInt(365 * 24 * 60 * 60),
      cliffDuration: 0,
    })
    .withGovernance({ type: 'default' })
    .withMigration({ type: 'uniswapV2' })
    .withUserAddress(account.address)
    .build();

  const result = await sdk.factory.createStaticAuction(params);

  console.log('Pool:', result.poolAddress);
  console.log('Token:', result.tokenAddress);
}

main();
```

### With Uniswap V4 migration

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

  console.log('Pool:', result.poolAddress);
  console.log('Token:', result.tokenAddress);
}

main();
```

---

## Using raw ticks

```typescript
const params = sdk.buildStaticAuction()
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
  .poolByTicks({ startTick: 175000, endTick: 225000, fee: 10000 })
  .withGovernance({ type: 'default' })
  .withMigration({ type: 'uniswapV2' })
  .withUserAddress(account.address)
  .build();
```
