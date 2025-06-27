---
icon: square-terminal
description: Quoter Class Reference
---

# Quoter

The `ReadQuoter` class provides a read-only interface to the Uniswap V4 Quoter contract for getting price quotes without executing transactions.

## Constructor

```typescript
new ReadQuoter(quoteV4Address: Address, drift: Drift<ReadAdapter>)
```

**Parameters:**

- `quoteV4Address` - Contract address of the V4 Quoter
- `drift` - Drift instance for blockchain interaction (defaults to new instance)

## Methods

### quoteExactInputV4

Get a price quote for swapping an exact amount of input tokens.

```typescript
async quoteExactInputV4(
  params: FunctionArgs<V4QuoterABI, 'quoteExactInputSingle'>['params']
): Promise<FunctionReturn<V4QuoterABI, 'quoteExactInputSingle'>>
```

**Parameters:**

- `params` - Arguments for the quoteExactInputSingle contract method

**Returns:** Promise resolving to raw contract return values

### quoteExactOutputV4

Get a price quote for receiving an exact amount of output tokens.

```typescript
async quoteExactOutputV4(
  params: FunctionArgs<V4QuoterABI, 'quoteExactOutputSingle'>['params']
): Promise<FunctionReturn<V4QuoterABI, 'quoteExactOutputSingle'>>
```

**Parameters:**

- `params` - Arguments for the quoteExactOutputSingle contract method

**Returns:** Promise resolving to raw contract return values

## Example Usage

```typescript
import { ReadQuoter } from "doppler-v4-sdk";

// Create quoter instance
const quoter = new ReadQuoter("0x...");

// Get quote for exact input
const inputQuote = await quoter.quoteExactInputV4({
  tokenIn: "0x...",
  tokenOut: "0x...",
  amountIn: 1000000n,
  fee: 3000,
  sqrtPriceLimitX96: 0n,
});

// Get quote for exact output
const outputQuote = await quoter.quoteExactOutputV4({
  tokenIn: "0x...",
  tokenOut: "0x...",
  amountOut: 1000000n,
  fee: 3000,
  sqrtPriceLimitX96: 0n,
});
```

## Key Features

- **Price Quotes**: Get accurate price quotes for both exact input and output swaps
- **No Gas Required**: Simulate swap outcomes without executing transactions
- **V4 Compatible**: Designed specifically for Uniswap V4 pools
- **Type Safety**: Full TypeScript support with proper type definitions
