# Get started

### Installation&#x20;

```bash
npm install @whetstone-research/doppler-sdk viem
# or
yarn add @whetstone-research/doppler-sdk viem
# or
pnpm add @whetstone-research/doppler-sdk viem
```

### Quickstart

```typescript
import { DopplerSDK } from '@whetstone-research/doppler-sdk';
import { createPublicClient, createWalletClient, http } from 'viem';
import { base } from 'viem/chains';

// Set up viem clients
const publicClient = createPublicClient({
  chain: base,
  transport: http(),
});

const walletClient = createWalletClient({
  chain: base,
  transport: http(),
  account: '0x...', // Your wallet address
});

// Initialize the SDK
const sdk = new DopplerSDK({
  publicClient,
  walletClient,
  chainId: base.id,
});
```

### Create your first Doppler token & auction :tada:

```typescript
import { DynamicAuctionBuilder } from '@whetstone-research/doppler-sdk'

const params = new DynamicAuctionBuilder()
  .tokenConfig({ name: 'My Token', symbol: 'MTK', tokenURI: 'https://example.com/metadata.json' })
  .saleConfig({ initialSupply: parseEther('1000000'), numTokensToSell: parseEther('900000'), numeraire: '0x...' })
  .poolConfig({ fee: 3000, tickSpacing: 60 })
  .auctionByTicks({
    durationDays: 7,
    epochLength: 3600,
    startTick: -92103,
    endTick: -69080,
    minProceeds: parseEther('100'),
    maxProceeds: parseEther('1000'),
    numPdSlugs: 5,
  })
  .withVesting({ duration: BigInt(365 * 24 * 60 * 60) })
  .withMigration({
    type: 'uniswapV4',
    fee: 3000,
    tickSpacing: 60,
    streamableFees: {
      lockDuration: 365 * 24 * 60 * 60,
      beneficiaries: [
        { address: '0x...', percentage: 5000 },
        { address: '0x...', percentage: 5000 },
      ],
    },
  })
  .withUserAddress('0x...')
  .build()

const result = await sdk.factory.createDynamicAuction(params)
console.log('Hook address:', result.hookAddress)
console.log('Token address:', result.tokenAddress)
```
