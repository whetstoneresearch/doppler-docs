---
description: >-
  Create coins with Doppler Multicurve for more granular supply curves
---

# Multicurve

## Using market cap ranges

```typescript
import { DopplerSDK } from '@whetstone-research/doppler-sdk';
import { parseEther, createPublicClient, createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { baseSepolia } from 'viem/chains';

const privateKey = process.env.PRIVATE_KEY as `0x${string}`;
const rpcUrl = process.env.RPC_URL ?? baseSepolia.rpcUrls.default.http[0];

async function main() {
  const account = privateKeyToAccount(privateKey);

  const publicClient = createPublicClient({
    chain: baseSepolia,
    transport: http(rpcUrl),
  });

  const walletClient = createWalletClient({
    chain: baseSepolia,
    transport: http(rpcUrl),
    account,
  });

  const sdk = new DopplerSDK({
    publicClient,
    walletClient,
    chainId: baseSepolia.id,
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
          shares: parseEther('0.3'),  // 30%
        },
        {
          marketCap: { start: 1_000_000, end: 5_000_000 },
          numPositions: 15,
          shares: parseEther('0.4'),  // 40%
        },
        {
          marketCap: { start: 4_000_000, end: 6_000_000 },
          numPositions: 10,
          shares: parseEther('0.2'),  // 20%
        },
        // Tail curve: extends liquidity to infinity
        {
          marketCap: { start: 6_000_000, end: 'max' },
          numPositions: 1,
          shares: parseEther('0.1'),  // 10% (completes 100%)
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

  console.log('Pool ID:', result.poolId);
  console.log('Token:', result.tokenAddress);
}

main();
```

### Curve rules

* First curve's `marketCap.start` = the launch price
* Curves must be contiguous or overlapping (no gaps)
* Shares must sum to exactly 1e18 (100%)
* **Tail curve (recommended)**: A final curve with `end: 'max'` and `numPositions: 1` ensures price continuity beyond the bonding curve. The tail's `start` must match the previous curve's `end`.

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
