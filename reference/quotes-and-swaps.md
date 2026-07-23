---
icon: rotate
---

# Quotes & swaps

Use the Doppler SDK to discover a pool and quote a swap. Then pass the resulting pool, direction, and amounts to Uniswap's [Universal Router SDK](https://github.com/Uniswap/sdks/tree/main/sdks/universal-router-sdk) to construct and execute the transaction.

This guide focuses on the information Doppler provides for:

* **Dynamic auctions**
* **Multicurve**
* **Multicurve Rehype**

All three are Uniswap V4 pools. They use the same quoting API but expose their initialized `PoolKey` differently.

## Install

```bash
pnpm add @whetstone-research/doppler-sdk viem
```

## Set up the SDK

Only a public client is required to discover pools and request quotes:

```ts
import {
  DopplerSDK,
  getAddresses,
} from '@whetstone-research/doppler-sdk/evm'
import { createPublicClient, http } from 'viem'
import { base } from 'viem/chains'

const publicClient = createPublicClient({
  chain: base,
  transport: http(rpcUrl),
})

const sdk = new DopplerSDK({
  publicClient,
  chainId: base.id,
})

const addresses = getAddresses(base.id)
```

Use `addresses.universalRouter` when executing the swap. It is the Doppler-compatible Universal Router for the selected chain.

Use Universal Router version:

* `2.1.1` on Robinhood (chain ID `4663`)
* `2.0` on every other supported network

{% hint style="warning" %}
Do not substitute a same-chain Universal Router from another address registry. Each Universal Router deployment is connected to a specific V4 `PoolManager`. Another router on the same chain may use a different `PoolManager`, where the Doppler pool is not initialized.
{% endhint %}

## Get the V4 `PoolKey`

The `PoolKey` contains the exact currencies, fee configuration, tick spacing, and hook address of an initialized V4 pool. Use the key returned from onchain state without reconstructing or modifying it.

### Multicurve and Multicurve Rehype

Look up either pool type by its Doppler asset address:

```ts
const pool = await sdk.getMulticurvePool(assetAddress)
const poolState = await pool.getState()

const poolKey = poolState.poolKey
```

For a buy, the input is normally the pool's numeraire:

```ts
const currencyInAddress = poolState.numeraire
```

For a sell, the input is the Doppler asset:

```ts
const currencyInAddress = poolState.asset
```

Use `poolState.poolKey` unchanged for both Multicurve and Multicurve Rehype pools.

### Dynamic auctions

A Dynamic auction stores its initialized `PoolKey` on its hook:

```ts
import {
  dopplerHookAbi,
  normalizePoolKey,
} from '@whetstone-research/doppler-sdk/evm'

const storedPoolKey = await publicClient.readContract({
  address: hookAddress,
  abi: dopplerHookAbi,
  functionName: 'poolKey',
})

const poolKey = normalizePoolKey(storedPoolKey)
```

The returned key already contains the Dynamic auction's fee flag, tick spacing, currencies, and hook address.

## Determine the swap direction

Uniswap V4 uses `zeroForOne` to identify the input side:

* `true`: swap `currency0` for `currency1`
* `false`: swap `currency1` for `currency0`

Derive it from the requested input currency:

```ts
function sameAddress(left: string, right: string) {
  return left.toLowerCase() === right.toLowerCase()
}

const inputIsCurrency0 = sameAddress(
  currencyInAddress,
  poolKey.currency0,
)
const inputIsCurrency1 = sameAddress(
  currencyInAddress,
  poolKey.currency1,
)

if (!inputIsCurrency0 && !inputIsCurrency1) {
  throw new Error('Input currency is not part of this PoolKey')
}

const zeroForOne = inputIsCurrency0
const currencyOutAddress = zeroForOne
  ? poolKey.currency1
  : poolKey.currency0
```

V4 represents native currency as the zero address. Wrapped native currency uses its ERC-20 address.

## Quote an exact-input swap

Use base units for `amountIn`:

```ts
const quote = await sdk.quoter.quoteExactInputV4({
  poolKey,
  zeroForOne,
  exactAmount: amountIn,
  hookData: '0x',
})

console.log('Expected output:', quote.amountOut)
console.log('Quoter gas estimate:', quote.gasEstimate)
```

For example, use `parseUnits('1', decimals)` for an ERC-20 or `parseEther('0.01')` for an 18-decimal native or wrapped-native input.

Select the Universal Router SDK encoding version for the current chain:

```ts
const universalRouterVersion =
  publicClient.chain.id === 4_663 ? '2.1.1' : '2.0'
```

The values required for a single-pool exact-input V4 swap are now:

```ts
const swapQuote = {
  poolKey,
  zeroForOne,
  currencyInAddress,
  currencyOutAddress,
  amountIn,
  amountOut: quote.amountOut,
  hookData: '0x',
  universalRouter: addresses.universalRouter,
  universalRouterVersion,
}
```

Apply your application's slippage policy to the quote when constructing the router transaction.

## Quote an exact-output swap

To calculate the input required for an exact output:

```ts
const exactOutputQuote = await sdk.quoter.quoteExactOutputV4({
  poolKey,
  zeroForOne,
  exactAmount: amountOut,
  hookData: '0x',
})

console.log('Required input:', exactOutputQuote.amountIn)
console.log('Quoter gas estimate:', exactOutputQuote.gasEstimate)
```

Apply your application's slippage policy to `exactOutputQuote.amountIn` when determining the maximum input.

## Execute with Uniswap Universal Router

Use the values above with Uniswap's official [Universal Router SDK](https://github.com/Uniswap/sdks/tree/main/sdks/universal-router-sdk). Its documentation covers transaction construction, currency objects, Permit2 or token approvals, slippage protection, native value, simulation, and submission.

When building the transaction:

* Use `addresses.universalRouter`.
* Select version `2.1.1` on Robinhood (chain ID `4663`).
* Select version `2.0` on every other supported network.
* Pass the exact `poolKey` returned by the Doppler SDK.
* Pass the derived `zeroForOne` value.
* Use empty hook data (`0x`) for these swaps.
* Simulate the final transaction before submitting it.

## V2 and V3 quotes

V2 and V3 are relevant for migrated pools and older Doppler deployments. New Dynamic, Multicurve, and Multicurve Rehype integrations should use the V4 path above.

### V3 exact-input quote

```ts
const quote = await sdk.quoter.quoteExactInputV3({
  tokenIn: tokenInAddress,
  tokenOut: tokenOutAddress,
  amountIn,
  fee: 3_000,
  sqrtPriceLimitX96: 0n,
})

console.log('Expected output:', quote.amountOut)
```

Use the initialized V3 pool's actual fee tier.

### V2 exact-input quote

```ts
const amounts = await sdk.quoter.quoteExactInputV2({
  amountIn,
  path: [tokenInAddress, tokenOutAddress],
})

const amountOut = amounts.at(-1)
if (amountOut === undefined) {
  throw new Error('V2 quoter returned an empty path')
}
```

Use the resulting route and quote with the corresponding V2 or V3 operation in the Universal Router SDK.

## Quote troubleshooting

**The pool cannot be quoted.** Read the `PoolKey` from `getState()` or the Dynamic hook and use it unchanged. Do not guess the fee, tick spacing, currency order, or hook address.

**The quoter returns `NotEnoughLiquidity`.** Verify `currencyInAddress`, `zeroForOne`, amount units, and current pool state. The requested amount may also be outside the pool's active liquidity.

**The quote succeeds but transaction construction or execution fails.** Verify that the Universal Router integration uses `getAddresses(chainId).universalRouter`. Use version `2.1.1` on Robinhood and version `2.0` on every other supported network. Then refer to the Universal Router SDK documentation for approvals, encoding, and execution.
