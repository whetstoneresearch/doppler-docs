---
description: >-
  Create Multicurve pools with RehypeDopplerHook for advanced fee distribution and buyback mechanisms
---

# Rehype Pools

Rehype Pools use the **RehypeDopplerHook** to distribute trading fees across beneficiaries, LPs, and buyback destinations.

## Fee distribution model

Fees are split into four categories (must sum to 100%):

| Category | Description |
|----------|-------------|
| **Asset Buyback** | Buy back the token |
| **Numeraire Buyback** | Buy back the quote token |
| **Beneficiary** | Stream to configured addresses |
| **LP** | Distribute to liquidity providers |

---

## Basic example

```typescript
import { DopplerSDK, getAddresses } from '@whetstone-research/doppler-sdk';
import { parseEther, createPublicClient, createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';

const privateKey = process.env.PRIVATE_KEY as `0x${string}`;
const rpcUrl = process.env.RPC_URL ?? 'https://mainnet.base.org';

async function main() {
  const account = privateKeyToAccount(privateKey);
  const addresses = getAddresses(base.id);

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

  // Get protocol owner for beneficiary requirement (min 5%)
  const protocolOwner = await sdk.getAirlockBeneficiary();

  // Beneficiaries receive the beneficiaryPercentWad portion of fees
  const beneficiaries = [
    protocolOwner, // 5% to protocol (minimum required)
    { beneficiary: account.address, shares: parseEther('0.95') }, // 95% to creator
  ];

  const BUYBACK_DESTINATION = account.address; // receives bought-back tokens

  const params = sdk
    .buildMulticurveAuction()
    .tokenConfig({
      name: 'Rehype Token',
      symbol: 'RHT',
      tokenURI: 'https://example.com/metadata.json',
    })
    .saleConfig({
      initialSupply: parseEther('1000000000'),
      numTokensToSell: parseEther('1000000000'),
      numeraire: addresses.weth,
    })
    .withCurves({
      numerairePrice: 3000, // ETH = $3000 USD
      curves: [
        { marketCap: { start: 500_000, end: 1_500_000 }, numPositions: 10, shares: parseEther('0.3') },
        { marketCap: { start: 1_000_000, end: 5_000_000 }, numPositions: 15, shares: parseEther('0.4') },
        { marketCap: { start: 4_000_000, end: 50_000_000 }, numPositions: 10, shares: parseEther('0.3') },
      ],
      beneficiaries,
    })
    .withRehypeDopplerHook({
      hookAddress: addresses.rehypeDopplerHook!,
      buybackDestination: BUYBACK_DESTINATION,
      customFee: 3000, // 0.3% swap fee
      // Distribution must sum to WAD (100%)
      assetBuybackPercentWad: parseEther('0.20'),      // 20%
      numeraireBuybackPercentWad: parseEther('0.20'), // 20%
      beneficiaryPercentWad: parseEther('0.30'),       // 30%
      lpPercentWad: parseEther('0.30'),                // 30%
    })
    .withGovernance({ type: 'noOp' })
    .withMigration({ type: 'noOp' })
    .withUserAddress(account.address)
    .withDopplerHookInitializer(addresses.dopplerHookInitializer!)
    .withNoOpMigrator(addresses.noOpMigrator!)
    .build();

  const result = await sdk.factory.createMulticurve(params);

  console.log('Pool:', result.poolAddress);
  console.log('Token:', result.tokenAddress);
}

main();
```

---

## Fee distribution strategies

### Heavy buyback (price support)

```typescript
.withRehypeDopplerHook({
  hookAddress: addresses.rehypeDopplerHook!,
  buybackDestination: burnAddress,
  customFee: 5000, // 0.5%
  assetBuybackPercentWad: parseEther('0.50'),    // 50% to buy back tokens
  numeraireBuybackPercentWad: parseEther('0.10'),
  beneficiaryPercentWad: parseEther('0.20'),
  lpPercentWad: parseEther('0.20'),
})
```

### LP-first (attract liquidity)

```typescript
.withRehypeDopplerHook({
  hookAddress: addresses.rehypeDopplerHook!,
  buybackDestination: treasury,
  customFee: 3000,
  assetBuybackPercentWad: parseEther('0.10'),
  numeraireBuybackPercentWad: parseEther('0.10'),
  beneficiaryPercentWad: parseEther('0.20'),
  lpPercentWad: parseEther('0.60'),              // 60% to LPs
})
```

### Treasury-building

```typescript
.withRehypeDopplerHook({
  hookAddress: addresses.rehypeDopplerHook!,
  buybackDestination: treasury,
  customFee: 3000,
  assetBuybackPercentWad: parseEther('0.10'),
  numeraireBuybackPercentWad: parseEther('0.40'), // 40% to treasury
  beneficiaryPercentWad: parseEther('0.40'),      // 40% to beneficiaries
  lpPercentWad: parseEther('0.10'),
})
```

---

## Collecting fees

Anyone can trigger fee collection; fees are distributed automatically:

```typescript
const pool = await sdk.getMulticurvePool(assetAddress);

const { fees0, fees1, transactionHash } = await pool.collectFees();
console.log('Fees collected (token0):', fees0);
console.log('Fees collected (token1):', fees1);
```

---

## Configuration reference

### RehypeDopplerHookConfig

| Parameter | Type | Description |
|-----------|------|-------------|
| `hookAddress` | `Address` | Deployed RehypeDopplerHook (must be whitelisted) |
| `buybackDestination` | `Address` | Receives buyback tokens |
| `customFee` | `number` | Swap fee in bps (3000 = 0.3%) |
| `assetBuybackPercentWad` | `bigint` | % for asset buyback (in WAD) |
| `numeraireBuybackPercentWad` | `bigint` | % for numeraire buyback (in WAD) |
| `beneficiaryPercentWad` | `bigint` | % for beneficiaries (in WAD) |
| `lpPercentWad` | `bigint` | % for LPs (in WAD) |
| `graduationCalldata` | `Hex` | Optional calldata on graduation |

---

## Rules

* **Distribution sum**: All four percentages must equal `WAD` (1e18)
* **Beneficiary shares**: Must sum to `WAD`; protocol owner needs at least 5%
* **Hook whitelisting**: `hookAddress` must be enabled in `DopplerHookInitializer`
* **Migration**: Use `noOp` - rehype pools don't migrate liquidity
* **Pool status**: Enters "Locked" (status = 2)
