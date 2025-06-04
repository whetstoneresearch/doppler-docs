---
icon: square-terminal
description: Quoter Class Reference
---

## Quoter Overview

The Quoter provides price quoting functionality for both Uniswap V3 and V2 swaps. It enables simulation of exact input/output swaps without executing transactions, with proper decimal handling for amounts.

### ReadQuoter

```typescript
export class ReadQuoter {
  constructor(
    quoteV2Address: Address,
    univ2RouterAddress: Address,
    drift: Drift<ReadAdapter> = createDrift()
  )
```

Methods:

- **quoteExactInputV3** - Get price quote for exact input swap (Uniswap V3)

```typescript
async quoteExactInputV3(
  params: FunctionArgs<QuoterV2ABI, "quoteExactInputSingle">["params"]
): Promise<FunctionReturn<QuoterV2ABI, "quoteExactInputSingle">>
```

- **quoteExactOutputV3** - Get price quote for exact output swap (Uniswap V3)

```typescript
async quoteExactOutputV3(
  params: FunctionArgs<QuoterV2ABI, "quoteExactOutputSingle">["params"]
): Promise<FunctionReturn<QuoterV2ABI, "quoteExactOutputSingle">>
```

- **quoteExactInputV2** - Get price quote for exact input swap (Uniswap V2)

```typescript
async quoteExactInputV2(
  params: FunctionArgs<UniswapV2Router02ABI, "getAmountsOut">
): Promise<FunctionReturn<UniswapV2Router02ABI, "getAmountsOut">>
```

- **quoteExactOutputV2** - Get price quote for exact output swap (Uniswap V2)

```typescript
async quoteExactOutputV2(
  params: FunctionArgs<UniswapV2Router02ABI, "getAmountsIn">
): Promise<FunctionReturn<UniswapV2Router02ABI, "getAmountsIn">>
```

#### Contract ABIs

```typescript
export type QuoterV2ABI = typeof quoterV2Abi;
export type UniswapV2Router02ABI = typeof uniswapV2Router02Abi;
```

### Example Usage

```typescript
const quoter = new ReadQuoter("0xQuoterV2Address", "0xUniV2RouterAddress");

// V3 Exact Input Quote
const v3InputQuote = await quoter.quoteExactInputV3({
  tokenIn: "0x...",
  tokenOut: "0x...",
  amountIn: 1000000n,
  fee: 3000,
  sqrtPriceLimitX96: 0n,
});

// V2 Exact Output Quote
const v2OutputQuote = await quoter.quoteExactOutputV2({
  amountOut: 500000n,
  path: ["0x...", "0x..."],
  to: "0x...",
  deadline: Math.floor(Date.now() / 1000) + 300,
});
```