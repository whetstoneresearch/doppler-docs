---
icon: square-terminal
description: Token Class Reference
---

## Token Class Overview

The `Token` class is a javascript class representation of a DERC20 token deployed by the airlock contract. It is responsible for serving information about the state of the token, and for performing various actions on Doppler tokens.

## ReadDerc20

Read operations for Doppler tokens.

```typescript
const token = new ReadDerc20(address: Address, drift?: Drift<ReadAdapter>);
```

Methods:

DERC20 tokens extend the ERC20 interface, and so have the following methods:

- `getName(): Promise<string>`
- `getSymbol(): Promise<string>`
- `getDecimals(): Promise<number>`
- `getTokenURI(): Promise<string>`
- `getAllowance(owner: Address, spender: Address): Promise<bigint>`
- `getBalanceOf(account: Address): Promise<bigint>`
- `getTotalSupply(): Promise<bigint>`

In addition, DERC20 tokens have the following methods:

- `getPool(): Promise<Address>`
  - `getPool` returns the address of the Uniswap V2 pool that the DERC20 token is migrated to after the liquidity bootstrapping process is complete.
- `getIsPoolUnlocked(): Promise<boolean>`
  - `getIsPoolUnlocked` returns `true` if the Uniswap V2 pool is unlocked, and `false` otherwise.
- `getVestingData(account: Address): Promise<{totalAmount: bigint, releasedAmount: bigint}>`
  - `getVestingData` returns the total amount of tokens that have been vested for a given account, and the amount of tokens that have been released to the account.
- `getVestingDuration(): Promise<bigint>`
  - `getVestingDuration` returns the duration of the vesting period for the token.
- `getVestingStart(): Promise<bigint>`
  - `getVestingStart` returns the start time of the vesting period for the token.
- `getVestedTotalAmount(): Promise<bigint>`
  - `getVestedTotalAmount` returns the total amount of tokens that have been vested for the token.
- `getYearlyMintRate(): Promise<bigint>`
  - `getYearlyMintRate` returns the yearly mint rate for the token.

## ReadWriteDerc20

Extends ReadDerc20 with basic write operations.

```typescript
const token = new ReadWriteDerc20(address: Address, drift: Drift<ReadWriteAdapter>);
```

Methods:

- `approve(spender: Address, value: bigint): Promise<Hash>`