---
description: >-
  Create coins with Doppler Multicurve for richer liquidity distributions
---

# Multicurve

## Using market cap ranges

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
    .buildMulticurveAuction()
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
    .withCurves({
      numerairePrice: 3000, // ETH = $3000 USD
      curves: [
        {
          marketCap: { start: 500_000, end: 1_500_000 },
          numPositions: 10,
          shares: parseEther('0.3'),
        },
        {
          marketCap: { start: 1_000_000, end: 5_000_000 },
          numPositions: 15,
          shares: parseEther('0.4'),
        },
        {
          marketCap: { start: 4_000_000, end: 50_000_000 },
          numPositions: 10,
          shares: parseEther('0.3'),
        },
      ],
    })
    .withVesting({
      duration: BigInt(365 * 24 * 60 * 60),
      cliffDuration: 0,
    })
    .withGovernance({ type: 'noOp' })
    .withMigration({ type: 'uniswapV2' })
    .withUserAddress(account.address)
    .build();

  const result = await sdk.factory.createMulticurve(params);

  console.log('Pool:', result.poolAddress);
  console.log('Token:', result.tokenAddress);
}

main();
```

### Curve rules

* First curve's `marketCap.start` = the launch price
* Curves must be contiguous or overlapping (no gaps)
* Shares must sum to exactly 1e18 (100%)

---

## Using raw ticks

```typescript
import { DopplerSDK, WAD } from '@whetstone-research/doppler-sdk';
import { parseEther } from 'viem';

const params = sdk
  .buildMulticurveAuction()
  .tokenConfig({ name: 'My Token', symbol: 'MTK', tokenURI: 'https://example.com/token.json' })
  .saleConfig({
    initialSupply: 1_000_000_000n * WAD,
    numTokensToSell: 900_000_000n * WAD,
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
  .withGovernance({ type: 'noOp' })
  .withMigration({ type: 'uniswapV2' })
  .withUserAddress(account.address)
  .build();
```
