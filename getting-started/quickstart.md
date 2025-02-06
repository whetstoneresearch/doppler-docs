---
icon: bullseye-arrow
description: Get started with the Doppler TypeScript SDK.
---

# Get started

{% hint style="info" %}
The [Doppler-SDK is open source on the Whetstone GitHub](https://github.com/whetstoneresearch/doppler-sdk). Contributions encouraged!
{% endhint %}

### Installation

```
# Using npm
npm install doppler-v3-sdk

# Using bun
bun add doppler-v3-sdk
```

### Swapping tokens

```
import { ReadQuoter, fixed } from "doppler-v3-sdk";

const quoter = new ReadQuoter("0x...quoterAddress");
const amountIn = fixed(1.5, 18); // 1.5 tokens with 18 decimals

const quote = await quoter.quoteExactInput(
  {
    tokenIn: "0x...",
    tokenOut: "0x...",
    amountIn: amountIn.toBigInt(),
    fee: 3000,
  },
  { tokenDecimals: 18, formatDecimals: 4 }
);

console.log(`Expected output: ${quote.formattedAmountOut}`);
```

### Quote prices

```
const quoter = new ReadQuoter(quoterAddress);
const quote = await quoter.quoteExactInput({
  params: {
    tokenIn: "0x...",
    tokenOut: "0x...",
    amountIn: parseUnits("1", 18),
    fee: 3000,
  },
  options: {
    tokenDecimals: 18,
    formatDecimals: 4,
  },
});
```

### Query Uniswap v3 pool data

```
import { ReadUniswapV3Pool } from "doppler-v3-sdk";

const pool = new ReadUniswapV3Pool("0x...poolAddress");
const [slot0, liquidityEvents] = await Promise.all([
  pool.getSlot0(),
  pool.getMintEvents(),
]);

console.log(`Current price: ${slot0.sqrtPriceX96}`);
console.log(`${liquidityEvents.length} liquidity positions found`);
```
