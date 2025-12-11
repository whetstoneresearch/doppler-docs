---
description: Create your first token with Doppler in as little as 10 minutes
hidden: true
icon: square-code
---

# Get started

Doppler has an open source TypeScript SDK that is officially supported.&#x20;

You can install it using your favorite package manager.

### Installation&#x20;

```bash
npm install @whetstone-research/doppler-sdk viem
# or
yarn add @whetstone-research/doppler-sdk viem
# or
pnpm add @whetstone-research/doppler-sdk viem
```

### Quickstart

Start by setting up viem, a popular library used for interacting with blockchain networks.&#x20;

Next, create a client for your local wallet using viem.&#x20;

Lastly, initialize the Doppler SDK utilizing the viem network connection and your local wallet.&#x20;

```typescript
import { DopplerSDK } from '@whetstone-research/doppler-sdk';
import { createPublicClient, createWalletClient, http } from 'viem';
import { base } from 'viem/chains';

// 1. Set up viem clients
const publicClient = createPublicClient({
  chain: base,
  transport: http(),
});

// 2. Set up your local wallet client
const walletClient = createWalletClient({
  chain: base,
  transport: http(),
  account: '0x...', // Your wallet address
});

// 3. Initialize the SDK
const sdk = new DopplerSDK({
  publicClient,
  walletClient,
  chainId: base.id,
});
```

### Create your first token with Doppler  :tada:

Use Doppler Multcurve to execute a customized token launch.&#x20;

```typescript
import { MulticurveBuilder } from '@whetstone-research/doppler-sdk'
import { parseEther } from 'viem'
import { base } from 'viem/chains'

const params = new MulticurveBuilder(base.id)
  .tokenConfig({ name: 'TEST', symbol: 'TEST', tokenURI: 'https://example.com/metadata.json' })
  .saleConfig({ initialSupply: parseEther('1000000'), numTokensToSell: parseEther('900000'), numeraire: '0x4200000000000000000000000000000000000006' })
  .withMulticurveAuction({
    fee: 0, // no fees
    tickSpacing: 8,
    curves: [
      { tickLower: 0, tickUpper: 240000, numPositions: 10, shares: parseEther('0.5') },
      { tickLower: 16000, tickUpper: 240000, numPositions: 10, shares: parseEther('0.5') },
    ],
  })
  .withGovernance({ type: 'noOp' }) // no governance
  .withMigration({ type: 'noOp' }) // no migration 
  .withUserAddress('0x...') // add your address
  .build()

const result = await sdk.factory.createMulticurve(params)
console.log('Pool address:', result.poolAddress)
console.log('Token address:', result.tokenAddress)
```

### Example configuration details

Here's is some context about what was configured with this launch.&#x20;

* Token config - metadata such as the name, symbol, and other relevant data like an image
* Sale config - allocations of the token and how much is available for sale&#x20;
* Fees - earnings on each swap in the pool created by Doppler. Set to 0 means no fees
* Curves - There were only two price curves defined, each with 10 unique liquidity positions&#x20;
* Governance & Migration - Using the `noOp` smart contract patterns for governance means that there is no token holder governance associated with this test token. Using the same `noOp` pattern for migration means that this token and it's generated liquidity will always "live" in the same Uniswap v4 pool that Doppler Multicurve created, for maximum simplicity.&#x20;

### Next steps

Continue learning more about Doppler and follow along with other [examples](sdk/examples/).&#x20;
