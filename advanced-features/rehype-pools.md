---
description: >-
  Create Multicurve pools with RehypeDopplerHook for advanced fee distribution
  and buyback mechanisms
icon: sack-dollar
---

# Fee Rehypothecation

Rehype Pools use the **RehypeDopplerHookInitializer** to distribute trading fees across beneficiaries, LPs, and buyback destinations.

## Fee distribution model

Fees are split into four categories (must sum to 100%):

| Category              | Description                       |
| --------------------- | --------------------------------- |
| **Asset Buyback**     | Buy back the token                |
| **Numeraire Buyback** | Buy back the quote token          |
| **Beneficiary**       | Stream to configured addresses    |
| **LP**                | Distribute to liquidity providers |

***

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
      hookAddress: addresses.rehypeDopplerHookInitializer!,
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

  console.log('Pool:', result.poolId);
  console.log('Token:', result.tokenAddress);
}

main();
```

***

## Fee distribution strategies

### Heavy buyback (price support)

```typescript
.withRehypeDopplerHook({
  hookAddress: addresses.rehypeDopplerHookInitializer!,
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
  hookAddress: addresses.rehypeDopplerHookInitializer!,
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
  hookAddress: addresses.rehypeDopplerHookInitializer!,
  buybackDestination: treasury,
  customFee: 3000,
  assetBuybackPercentWad: parseEther('0.10'),
  numeraireBuybackPercentWad: parseEther('0.40'), // 40% to treasury
  beneficiaryPercentWad: parseEther('0.40'),      // 40% to beneficiaries
  lpPercentWad: parseEther('0.10'),
})
```

***

## Configuring and updating fee distribution

After configuring the token, sale, and curves, set the Rehype fee beneficiaries and an optional durable fee controller. Beneficiaries receive the config's beneficiary allocations according to their shares. The controller can replace the config after launch, or it can be assigned to addresses such as `0xdead` to make the config immutable.

```typescript
builder
  .withRehypeDopplerHookInitializer({
    hookAddress: addresses.rehypeDopplerHookInitializer!,
    startFee: 500_000,
    endFee: 12_000,
    durationSeconds: 20,
    feeRoutingMode: 'routeToBeneficiaryFees',
    feeBeneficiaries: [
      { beneficiary: treasury, shares: parseEther('0.20') },
      { beneficiary: account.address, shares: parseEther('0.80') },
    ],
    feeDistributionInfo: {
      assetFeesToAssetBuybackWad: 0n,
      assetFeesToNumeraireBuybackWad: parseEther('1'),
      assetFeesToBeneficiaryWad: 0n,
      assetFeesToLpWad: 0n,
      numeraireFeesToAssetBuybackWad: 0n,
      numeraireFeesToNumeraireBuybackWad: 0n,
      numeraireFeesToBeneficiaryWad: parseEther('1'),
      numeraireFeesToLpWad: 0n,
    },
  })
  .withFeeDistributionController(controller);
```

Beneficiary shares must sum to `WAD`. The controller is fixed at pool creation, so use an operational wallet or multisig that will remain available to sign updates.

`buybackDestination` is an alternative builder input for the same overloaded onchain `buybackDst` field, so do not set both. That field receives proceeds when no `feeBeneficiaries` are configured and/or the `directBuyback` routing mode is used.

Connect the SDK to the controller wallet to replace the pool's complete fee distribution config:

```typescript
const hook = await sdk.getRehypeDopplerHookInitializer(
  addresses.rehypeDopplerHookInitializer!,
);

const { transactionHash } = await hook.setFeeDistribution(poolId, {
  assetFeesToAssetBuybackWad: 0n,
  assetFeesToNumeraireBuybackWad: 0n,
  assetFeesToBeneficiaryWad: parseEther('1'),
  assetFeesToLpWad: 0n,
  numeraireFeesToAssetBuybackWad: parseEther('1'),
  numeraireFeesToNumeraireBuybackWad: 0n,
  numeraireFeesToBeneficiaryWad: 0n,
  numeraireFeesToLpWad: 0n,
});
```

`setFeeDistribution` replaces all eight values; it is not a partial update. The four `assetFeesTo*` values must sum to `WAD`, and the four `numeraireFeesTo*` values must separately sum to `WAD`. The update does not change the fee schedule, routing mode, or beneficiary shares.

Read the active config with:

```typescript
const distribution = await hook.getFeeDistributionInfo(poolId);
```

***

## Atomic dev buys

Use `withDevBuy` to create the pool and execute an exact-input purchase in the same transaction:

```typescript
const params = sdk
  .buildMulticurveAuction()
  // Configure the token, sale, curves, and Rehype hook as above.
  .withGovernance({ type: 'noOp' })
  .withMigration({ type: 'noOp' })
  .withDevBuy({
    exactAmountIn: parseEther('0.01'),
    recipient: account.address,
    vesting: {
      vestingDuration: 7n * 24n * 60n * 60n,
      cliffDuration: 24n * 60n * 60n,
      permissionlessClaim: false,
    },
  })
  .withUserAddress(account.address)
  .build();

const simulated = await sdk.factory.simulateCreateMulticurve(params);
console.log('Simulated output:', simulated.devBuy?.simulatedAmountOut);

const result = await simulated.execute();
console.log('Dev buy output:', result.devBuy?.amountOut);
```

The SDK uses the chain's configured Bundler by default. Use `.withBundler(address)` only for a compatible custom deployment.

For compatible Rehype pools, initialization grants that Bundler a pool-specific exemption for its first swap in the launch transaction. The dev buy still pays the Airlock owner's share of the Rehype fee, but bypasses all other Rehype fee portions, including beneficiary, buyback, and LP-reinvestment fees. It does not bypass the pool's Uniswap v4 LP fee. The exemption cannot be used by a normal router and expires when the transaction ends, every later swap uses the configured fee schedule.

Native numeraires send exactly `exactAmountIn` with the bundle. ERC-20 numeraires require sufficient balance and Bundler allowances. `createMulticurve` submits an approval first when needed.

Omit `vesting` to send purchased tokens directly to `recipient`. When supplied, `vestingDuration` must be at least one day, `cliffDuration` cannot exceed it, and permissionless claims can trigger delivery but cannot change the recipient.

***

## Collecting fees

Anyone can trigger fee collection; fees are distributed automatically to the initializer, but each address listed as a beneficiary must then claim their own fees from the initializer:

```typescript
const pool = await sdk.getMulticurvePool(assetAddress);

const { fees0, fees1, transactionHash } = await pool.collectFees();
console.log('Fees collected (token0):', fees0);
console.log('Fees collected (token1):', fees1);
```

***

## Configuration reference

### RehypeDopplerHookConfig

| Parameter                    | Type      | Description                                                 |
| ---------------------------- | --------- | ----------------------------------------------------------- |
| `hookAddress`                | `Address` | Deployed RehypeDopplerHookInitializer (must be whitelisted) |
| `buybackDestination`         | `Address` | Receives buyback tokens                                     |
| `customFee`                  | `number`  | Swap fee in bps (3000 = 0.3%)                               |
| `feeDistributionInfo`        | `object`  | Separate four-way WAD splits for asset and numeraire fees   |
| `assetBuybackPercentWad`     | `bigint`  | % for asset buyback (in WAD)                                |
| `numeraireBuybackPercentWad` | `bigint`  | % for numeraire buyback (in WAD)                            |
| `beneficiaryPercentWad`      | `bigint`  | % for beneficiaries (in WAD)                                |
| `lpPercentWad`               | `bigint`  | % for LPs (in WAD)                                          |
| `graduationCalldata`         | `Hex`     | Optional calldata on graduation                             |

***

## Rules

* **Distribution sum**: All four percentages must equal `WAD` (1e18)
* **Distribution updates**: Only the fixed fee distribution controller can replace the eight-value config
* **Beneficiary shares**: Must sum to `WAD`; protocol owner needs at least 5%
  * Note: each beneficiary must collect their own fees by calling collectFees
* **Hook whitelisting**: `hookAddress` must be enabled in `DopplerHookInitializer`
* **Migration**: Use `noOp` - rehype pools don't migrate liquidity
* **Dev buys**: Supported by compatible DopplerHookInitializer and Rehype initializer deployments
* **Pool status**: Enters "Locked" (status = 2)
