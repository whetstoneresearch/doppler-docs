---
icon: hand-wave
---

# Home

### About

[Doppler](https://doppler.lol) is an onchain protocol for launching tokens through various [price discovery auctions](https://aada.ms/pdfs/pda.pdf). Applications integrate Doppler, configure their token launch parameters, and get to market faster than building in-house smart contracts. Teams like [Zora](https://zora.co), [Paragraph](https://paragraph.com), and [Noice](https://noice.so) use Doppler to create new tokens by configuring inputs such as supply curves, vesting schedules, inflation mechanics, governance structures, and ongoing economics, such as fees and treasury management.

### Get started&#x20;

```bash
npm install @whetstone-research/doppler-sdk viem
```

```javascript
import { DopplerSDK } from '@whetstone-research/doppler-sdk';
import { createPublicClient, createWalletClient, http } from 'viem';
import { MulticurveBuilder } from '@whetstone-research/doppler-sdk'
import { parseEther } from 'viem'
import { base } from 'viem/chains'

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

// 4. Configure a multicurve auction 
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

With this [Doppler Multicurve](https://doppler.lol/multicurve.pdf) Launch, we configured:&#x20;

* Token config - metadata such as the name, symbol, and other relevant data like an image
* Sale config - allocations of the token and how much is available for sale&#x20;
* Fees - earnings on each swap in the pool created by Doppler. Set to 0 means no fees
* Curves - There were only two price curves defined, each with 10 unique liquidity positions&#x20;
* Governance & Migration - There is no governance or migration associated with this token&#x20;

### Next steps

Continue learning more about Doppler and follow along with other SDK [examples](sdk/examples/).&#x20;
