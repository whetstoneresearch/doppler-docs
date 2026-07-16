---
description: Create a Solana launch with a dynamic fee schedule
---

# Dynamic Fee Launch

Pass `dynamicFee` to `createLaunch` to configure a fee schedule on the Doppler dynamic fee hook. The hook normalizes the schedule during the `BEFORE_CREATE` callback and stores the normalized schedule in the launch hook payload.

This snippet assumes `payer` and `rpc` are already initialized with `@solana/kit`.

```typescript
import {
  generateKeyPairSigner,
  type Address,
} from '@solana/kit';
import {
  createLaunch,
  cpmmHook,
  initializer,
  deriveSolanaCpmmDeployment,
  DOPPLER_SOLANA_DEVNET_PROGRAM_ADDRESSES,
} from '@whetstone-research/doppler-sdk/solana';

const WSOL_MINT =
  'So11111111111111111111111111111111111111112' as Address;

const deployment = await deriveSolanaCpmmDeployment(
  DOPPLER_SOLANA_DEVNET_PROGRAM_ADDRESSES,
);

const baseMint = await generateKeyPairSigner();
const baseVault = await generateKeyPairSigner();
const quoteVault = await generateKeyPairSigner();
const namespace = payer.address;
const launchId = initializer.createLaunchId();

const addresses = await initializer.deriveCreateLaunchAddresses({
  deployment,
  namespace,
  launchId,
  baseMint,
});

const { instruction } = await createLaunch({
  deployment,
  namespace,
  launchId,
  addresses,
  launchAccounts: {
    baseMint,
    quoteMint: WSOL_MINT,
    baseVault,
    quoteVault,
  },
  payer,
  authority: payer,
  supply: {
    baseDecimals: 6,
    baseTotalSupply: 1_000_000_000n * 10n ** 6n,
    baseForDistribution: 0n,
    baseForLiquidity: 0n,
  },
  curve: {
    curveVirtualBase: 1_000_000_000n * 10n ** 6n,
    curveVirtualQuote: 10n * 1_000_000_000n,
    swapFeeBps: 200,
  },
  dynamicFee: {
    startingTime: 0n,
    startFeeBps: 8_000,
    endFeeBps: 200,
    durationSeconds: 10n * 60n,
  },
  migration: {
    minRaiseQuote: 50n * 1_000_000_000n,
  },
  metadata: null,
  feeBeneficiaries: [{ wallet: payer.address, shareBps: 10_000 }],
});
```

Submit the returned `instruction` with the generated mint and vault signers. The SDK sets the dynamic fee hook program, `HF_BEFORE_CREATE | HF_BEFORE_SWAP`, the 32-byte schedule payload, the create-hook account commitment, and the swap hook remaining-account commitment.

`startingTime: 0n` means the hook should start the schedule at launch creation. During `initialize_launch`, the hook replaces it with the current Solana clock timestamp before the launch account is stored.

The effective swap fee is the greater of the current schedule fee and the launch's static `swapFeeBps`, so the schedule cannot reduce the fee below the static launch fee.

After sending the transaction, you can verify that the launch is using the dynamic fee hook:

```typescript
const launch = await initializer.fetchLaunch(rpc, addresses.launch, {
  programId: deployment.initializerProgram,
});

if (!launch) {
  throw new Error('Launch account not found');
}

const hookPayload = new Uint8Array(
  launch.hookPayload.bytes.slice(0, launch.hookPayload.len),
);

if (launch.hookProgram !== deployment.cpmmHookProgram) {
  throw new Error('Launch is not using the dynamic fee hook');
}

if (!cpmmHook.isDynamicFeeSchedulePayload(hookPayload)) {
  throw new Error('Launch hook payload does not contain a dynamic fee schedule');
}
```

To combine dynamic fees with cosigner gating, pass the same dynamic fee schedule plus `cosigner` and, optionally, `cosignGateExpiresAt`:

```typescript
const { instruction } = await createLaunch({
  // ...same launch inputs as above
  dynamicFee: {
    startingTime: 0n,
    startFeeBps: 8_000,
    endFeeBps: 200,
    durationSeconds: 10n * 60n,
  },
  cosigner,
  cosignGateExpiresAt,
});
```

With both features enabled, swaps require the configured cosigner until expiry, and the dynamic fee schedule continues to apply throughout the initializer trading phase. Omit `cosignGateExpiresAt` for an indefinite gate; in that case the schedule remains the entire hook payload and the presence of the cosigner config account enables gating.
