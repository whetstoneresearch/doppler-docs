---
icon: square-terminal
description: Quoter Class Reference
---

# Doppler Lens

The `ReadDopplerLens` class provides read-only access to the Doppler Lens contract, which fetches virtual updates to the Doppler Dutch auction that aren't reflected in the current chain state. This is essential for getting accurate real-time pricing and liquidity information during active price discovery phases.

## Overview

The Doppler Lens serves as a sophisticated quoter that can simulate swaps against Doppler pools and return detailed information about:

- Current pool state (price, tick, liquidity)
- Virtual token amounts in each position
- Position data for lower, upper, and price discovery slugs
- Real-time pricing without executing transactions

## Constructor

```typescript
new ReadDopplerLens(address: Hex, drift: Drift<ReadAdapter>)
```

**Parameters:**

- `address` - The address of the DopplerLensQuoter contract
- `drift` - Drift instance with read adapter (creates default if not provided)

## Core Methods

### poolManager

Retrieves the address of the Uniswap V4 pool manager used by the lens.

```typescript
async poolManager(): Promise<Address>
```

**Returns:** Promise resolving to the pool manager address

### stateView

Retrieves the address of the state view contract used for reading pool state.

```typescript
async stateView(): Promise<Address>
```

**Returns:** Promise resolving to the state view contract address

### quoteDopplerLensData

The main method for getting comprehensive Doppler pool data. This simulates a swap to extract current pool state and position information.

```typescript
async quoteDopplerLensData(
  params: QuoteExactSingleParams
): Promise<DopplerLensReturnData>
```

**Parameters:**

- `params` - Quote parameters containing:
  - `poolKey` - The pool identifier (tokens, fee, tick spacing, hooks)
  - `zeroForOne` - Direction of the swap (token0 → token1 or vice versa)
  - `exactAmount` - Amount to simulate swapping
  - `hookData` - Additional data for the hook (usually empty)

**Returns:** Promise resolving to:

- `sqrtPriceX96` - Current pool price in sqrt format
- `amount0` - Total amount of token0 across all positions
- `amount1` - Total amount of token1 across all positions
- `tick` - Current tick of the pool

## Types

### QuoteExactSingleParams

```typescript
interface QuoteExactSingleParams {
  poolKey: PoolKey;
  zeroForOne: boolean;
  exactAmount: bigint;
  hookData: Hex;
}
```

### PoolKey

```typescript
interface PoolKey {
  currency0: Address;
  currency1: Address;
  fee: number;
  tickSpacing: number;
  hooks: Address;
}
```

### DopplerLensReturnData

```typescript
interface DopplerLensReturnData {
  sqrtPriceX96: bigint;
  amount0: bigint;
  amount1: bigint;
  tick: number;
}
```

## Example Usage

```typescript
import { ReadDopplerLens } from "doppler-v4-sdk";

// Create lens instance
const lens = new ReadDopplerLens(lensAddress);

// Get pool manager and state view addresses
const poolManager = await lens.poolManager();
const stateView = await lens.stateView();

// Quote current pool state
const poolKey = {
  currency0: numeraireAddress,
  currency1: tokenAddress,
  fee: 20_000,
  tickSpacing: 2,
  hooks: dopplerHookAddress,
};

const quoteData = await lens.quoteDopplerLensData({
  poolKey,
  zeroForOne: true,
  exactAmount: 1, // Simulate swapping 1 wei
  hookData: "0x",
});

console.log("Current pool state:", {
  price: quoteData.sqrtPriceX96,
  tick: quoteData.tick,
  token0Amount: quoteData.amount0.toString(),
  token1Amount: quoteData.amount1.toString(),
});
```

## Use Cases

### Real-time Price Monitoring

```typescript
// Monitor price changes during Dutch auction
async function monitorPrice() {
  const data = await lens.quoteDopplerLensData({
    poolKey: myPoolKey,
    zeroForOne: true,
    exactAmount: parseEther("1"),
    hookData: "0x",
  });

  // Convert sqrt price to human-readable price
  const price = (Number(data.sqrtPriceX96) / 2 ** 96) ** 2;
  console.log(`Current price: ${price}`);
}
```

### Liquidity Analysis

```typescript
// Analyze total liquidity across all positions
async function analyzeLiquidity() {
  const data = await lens.quoteDopplerLensData({
    poolKey: myPoolKey,
    zeroForOne: false,
    exactAmount: 1n, // Minimal amount for state query
    hookData: "0x",
  });

  console.log("Total liquidity:", {
    totalToken0: formatEther(data.amount0),
    totalToken1: formatEther(data.amount1),
    tick: data.tick,
  });
}
```

## Key Features

- **Real-time State**: Get current pool state without waiting for blockchain updates
- **Virtual Positions**: See combined liquidity across lower, upper, and price discovery positions
- **Simulation Based**: Uses revert-based simulation for gas-free queries
- **Price Discovery**: Essential for monitoring active Dutch auctions
- **Non-view Functions**: Handles complex state calculations that require simulation

## Technical Notes

The lens contract uses a revert-based approach where it:

1. Simulates a swap to update internal state
2. Calculates position data across all Doppler slugs (lower, upper, price discovery)
3. Reverts with the calculated data to return results
4. Parses the revert data to extract meaningful information

This approach allows complex calculations that wouldn't be possible with pure view functions while maintaining gas efficiency for off-chain queries.
