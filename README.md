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
import {
  createPublicClient,
  createWalletClient,
  http,
  parseEther
} from 'viem';
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

// 4. Configure a multicurve auction using market cap ranges
const WETH = '0x4200000000000000000000000000000000000006';

const params = sdk
  .buildMulticurveAuction()
  .tokenConfig({
    name: 'TEST',
    symbol: 'TEST',
    tokenURI: 'https://example.com/metadata.json'
  })
  .saleConfig({
    initialSupply: parseEther('1000000000'),
    numTokensToSell: parseEther('900000000'),
    numeraire: WETH,
  })
  .withCurves({
    numerairePrice: 3000, // ETH = $3000 USD
    curves: [
      {
        marketCap: { start: 500_000, end: 1_500_000 },
        numPositions: 10,
        shares: parseEther('0.4')
      },
      {
        marketCap: { start: 1_000_000, end: 5_000_000 },
        numPositions: 10,
        shares: parseEther('0.5')
      },
      {
        marketCap: { start: 5_000_000, end: 'max' },
        numPositions: 1,
        shares: parseEther('0.1')
      },
    ],
  })
  .withGovernance({ type: 'noOp' })
  .withMigration({ type: 'noOp' })
  .withUserAddress('0x...')
  .build()

const result = await sdk.factory.createMulticurve(params)
console.log('Pool:', result.poolId)
console.log('Token address:', result.tokenAddress)
```

### Example configuration details

With this [Doppler Multicurve](https://doppler.lol/multicurve.pdf) Launch, we configured:&#x20;

* Token config - metadata such as the name, symbol, and other relevant data like an image
* Sale config - allocations of the token and how much is available for sale&#x20;
* Curves - Two price curves defined by market cap ranges (USD), each with 10 unique liquidity positions. The first curve's start market cap ($500k) sets the launch price.
* Governance & Migration - There is no governance or migration associated with this token&#x20;

### Next steps

Continue learning more about Doppler and follow along with other SDK [examples](sdk/examples/).&#x20;
