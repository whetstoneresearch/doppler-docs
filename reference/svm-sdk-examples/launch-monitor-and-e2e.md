---
description: Create a Solana launch, buy from the curve, and migrate to CPMM
---

# Launch, monitor, and e2e

```typescript
/**
 * Example: Create, buy, and migrate a launch (Solana)
 *
 * Demonstrates:
 * - Creating a launch with CPMM migration
 * - Previewing and executing a curve buy
 * - Migrating the launch to CPMM
 */
import './env.js';

import {
  TOKEN_PROGRAM_ADDRESS,
  findAssociatedTokenPda,
  getCreateAssociatedTokenIdempotentInstruction,
  getSyncNativeInstruction,
} from '@solana-program/token';
import {
  SYSTEM_PROGRAM_ADDRESS,
  getTransferSolInstruction,
} from '@solana-program/system';
import {
  AccountRole,
  createKeyPairSignerFromBytes,
  createTransactionMessage,
  createSolanaRpc,
  createSolanaRpcSubscriptions,
  generateKeyPairSigner,
  getBase64EncodedWireTransaction,
  getSignatureFromTransaction,
  pipe,
  appendTransactionMessageInstructions,
  sendAndConfirmTransactionFactory,
  setTransactionMessageFeePayerSigner,
  setTransactionMessageLifetimeUsingBlockhash,
  signTransactionMessageWithSigners,
  type Address,
  type Instruction,
} from '@solana/kit';
import { SYSVAR_RENT_ADDRESS } from '@solana/sysvars';

import {
  cpmm,
  initializer,
  cpmmMigrator,
} from '@whetstone-research/doppler-sdk/solana';

// ============================================================================
// Environment
// ============================================================================

const keypairJson = process.env.SOLANA_KEYPAIR;
const rpcUrl = process.env.SOLANA_RPC_URL ?? 'https://api.devnet.solana.com';
const wsUrl = process.env.SOLANA_WS_URL ?? 'wss://api.devnet.solana.com';

if (!keypairJson) {
  throw new Error('SOLANA_KEYPAIR must be set (JSON array of 64 bytes)');
}

const WSOL_MINT: Address =
  'So11111111111111111111111111111111111111112' as Address;

// ============================================================================
// Main
// ============================================================================

async function main() {
  const payer = await createKeyPairSignerFromBytes(
    new Uint8Array(JSON.parse(keypairJson as string)),
  );

  const rpc = createSolanaRpc(rpcUrl);
  const rpcSubscriptions = createSolanaRpcSubscriptions(wsUrl);
  const sendAndConfirmTransaction = sendAndConfirmTransactionFactory({
    rpc,
    rpcSubscriptions,
  });

  async function send(instructions: Instruction[]) {
    const { value: latestBlockhash } = await rpc.getLatestBlockhash().send();
    const transactionMessage = pipe(
      createTransactionMessage({ version: 0 }),
      (tx) => setTransactionMessageFeePayerSigner(payer, tx),
      (tx) => setTransactionMessageLifetimeUsingBlockhash(latestBlockhash, tx),
      (tx) => appendTransactionMessageInstructions(instructions, tx),
    );

    const signedTransaction =
      await signTransactionMessageWithSigners(transactionMessage);
    await sendAndConfirmTransaction(
      signedTransaction as Parameters<typeof sendAndConfirmTransaction>[0],
      { commitment: 'confirmed' },
    );
    return getSignatureFromTransaction(signedTransaction);
  }

  // ── Step 1: Create launch ─────────────────────────────────────────────────
  const BASE_DECIMALS = 6;
  const BASE_TOTAL_SUPPLY = 1_000_000_000n * 10n ** BigInt(BASE_DECIMALS);
  const MIN_RAISE_QUOTE = 100_000_000n;

  const baseMint = await generateKeyPairSigner();
  const baseVault = await generateKeyPairSigner();
  const quoteVault = await generateKeyPairSigner();
  const metadataAccount = await initializer.getTokenMetadataAddress(
    baseMint.address,
  );

  const namespace = payer.address;
  const launchId = initializer.launchIdFromU64(BigInt(Date.now()));
  const [launch] = await initializer.getLaunchAddress(namespace, launchId);
  const [launchAuthority] = await initializer.getLaunchAuthorityAddress(launch);
  const [initializerConfig] = await initializer.getConfigAddress();
  const [cpmmConfig] = await cpmm.getConfigAddress();
  const [cpmmMigratorState] =
    await cpmmMigrator.getCpmmMigratorStateAddress(launch);

  const poolInit = await cpmm.getPoolInitAddresses(baseMint.address, WSOL_MINT);
  const pool = poolInit.pool[0];
  const poolAuthority = poolInit.authority[0];
  const poolVault0 = poolInit.vault0[0];
  const poolVault1 = poolInit.vault1[0];
  const protocolPosition = poolInit.protocolPosition[0];
  const [migrationAuthority] =
    await cpmmMigrator.getCpmmMigrationAuthorityAddress();
  const [launchLpPosition] = await cpmm.getPositionAddress(
    pool,
    launchAuthority,
    0n,
  );

  const [adminBaseAta] = await findAssociatedTokenPda({
    owner: payer.address,
    mint: baseMint.address,
    tokenProgram: TOKEN_PROGRAM_ADDRESS,
  });
  const [adminQuoteAta] = await findAssociatedTokenPda({
    owner: payer.address,
    mint: WSOL_MINT,
    tokenProgram: TOKEN_PROGRAM_ADDRESS,
  });

  const migratorInitCalldata = cpmmMigrator.encodeRegisterLaunchCalldata({
    cpmmConfig,
    initialSwapFeeBps: 100,
    initialFeeSplitBps: 5000,
    recipients: [],
    minRaiseQuote: MIN_RAISE_QUOTE,
    minMigrationPriceQ64Opt: null,
  });
  const migratorMigrateCalldata = cpmmMigrator.encodeMigrateCalldata({
    baseForDistribution: 0n,
    baseForLiquidity: 0n,
  });

  const migratorRemainingAccounts = [
    cpmmMigratorState,
    cpmmConfig,
    pool,
    poolAuthority,
    poolVault0,
    poolVault1,
    protocolPosition,
    launchLpPosition,
    cpmm.CPMM_PROGRAM_ID,
    migrationAuthority,
    adminBaseAta,
    adminQuoteAta,
  ];

  const createLaunchIx = await initializer.createInitializeLaunchInstruction(
    {
      config: initializerConfig,
      launch,
      launchAuthority,
      baseMint,
      quoteMint: WSOL_MINT,
      baseVault,
      quoteVault,
      payer,
      authority: payer,
      migratorProgram: cpmmMigrator.CPMM_MIGRATOR_PROGRAM_ID,
      cpmmConfig,
      baseTokenProgram: TOKEN_PROGRAM_ADDRESS,
      quoteTokenProgram: TOKEN_PROGRAM_ADDRESS,
      systemProgram: SYSTEM_PROGRAM_ADDRESS,
      rent: SYSVAR_RENT_ADDRESS,
      metadataAccount,
      addressLookupTable: initializer.DOPPLER_DEVNET_ALT,
    },
    {
      namespace,
      launchId,
      baseDecimals: BASE_DECIMALS,
      baseTotalSupply: BASE_TOTAL_SUPPLY,
      baseForDistribution: 0n,
      baseForLiquidity: 0n,
      curveVirtualBase: BASE_TOTAL_SUPPLY,
      curveVirtualQuote: 100_000_000n,
      curveFeeBps: 200,
      curveKind: initializer.CURVE_KIND_XYK,
      curveParams: new Uint8Array([initializer.CURVE_PARAMS_FORMAT_XYK_V0]),
      allowBuy: true,
      allowSell: true,
      sentinelProgram: initializer.CPMM_SENTINEL_PROGRAM_ID,
      sentinelFlags: initializer.SF_BEFORE_SWAP,
      sentinelCalldata: new Uint8Array(),
      migratorInitCalldata,
      migratorMigrateCalldata,
      sentinelRemainingAccountsHash: initializer.EMPTY_REMAINING_ACCOUNTS_HASH,
      migratorInitRemainingAccountsHash:
        initializer.computeRemainingAccountsHash([
          cpmmMigratorState,
          cpmmConfig,
        ]),
      migratorRemainingAccountsHash:
        initializer.computeRemainingAccountsHash(migratorRemainingAccounts),
      metadataName: 'E2E Token',
      metadataSymbol: 'E2E',
      metadataUri: 'https://example.com/e.json',
    },
  );

  console.log('Create launch:', await send([createLaunchIx]));
  console.log('Launch:', launch);
  console.log('Base mint:', baseMint.address);

  // ── Step 2: Preview and execute buy ───────────────────────────────────────
  const BUY_AMOUNT_IN = 200_000_000n;

  const previewIx = initializer.createPreviewSwapExactInInstruction(
    { launch, baseVault: baseVault.address, quoteVault: quoteVault.address },
    { amountIn: BUY_AMOUNT_IN, direction: initializer.DIRECTION_BUY },
  );

  const { value: previewBlockhash } = await rpc.getLatestBlockhash().send();
  const previewMessage = pipe(
    createTransactionMessage({ version: 0 }),
    (tx) => setTransactionMessageFeePayerSigner(payer, tx),
    (tx) => setTransactionMessageLifetimeUsingBlockhash(previewBlockhash, tx),
    (tx) => appendTransactionMessageInstructions([previewIx], tx),
  );
  const signedPreview = await signTransactionMessageWithSigners(previewMessage);
  const { value: simulateResult } = await rpc
    .simulateTransaction(getBase64EncodedWireTransaction(signedPreview), {
      encoding: 'base64',
      replaceRecentBlockhash: true,
    })
    .send();

  if (simulateResult.returnData?.data) {
    const returnBytes = Uint8Array.from(
      atob(simulateResult.returnData.data[0]),
      (c) => c.charCodeAt(0),
    );
    const preview = initializer.decodePreviewSwapExactInResult(returnBytes);
    console.log('Preview amount out:', preview.amountOut.toString());
  }

  const createBaseAtaIx = getCreateAssociatedTokenIdempotentInstruction({
    payer,
    ata: adminBaseAta,
    owner: payer.address,
    mint: baseMint.address,
  });
  const createQuoteAtaIx = getCreateAssociatedTokenIdempotentInstruction({
    payer,
    ata: adminQuoteAta,
    owner: payer.address,
    mint: WSOL_MINT,
  });
  const transferSolIx = getTransferSolInstruction({
    source: payer,
    destination: adminQuoteAta,
    amount: BUY_AMOUNT_IN + 2_039_280n,
  });
  const syncNativeIx = getSyncNativeInstruction({ account: adminQuoteAta });
  const buyIx = initializer.createCurveSwapExactInInstruction(
    {
      config: initializerConfig,
      launch,
      launchAuthority,
      baseVault: baseVault.address,
      quoteVault: quoteVault.address,
      userBaseAccount: adminBaseAta,
      userQuoteAccount: adminQuoteAta,
      baseMint: baseMint.address,
      quoteMint: WSOL_MINT,
      user: payer,
      sentinelProgram: initializer.CPMM_SENTINEL_PROGRAM_ID,
      baseTokenProgram: TOKEN_PROGRAM_ADDRESS,
      quoteTokenProgram: TOKEN_PROGRAM_ADDRESS,
    },
    {
      amountIn: BUY_AMOUNT_IN,
      minAmountOut: 1n,
      direction: initializer.DIRECTION_BUY,
    },
  );

  console.log(
    'Buy:',
    await send([
      createBaseAtaIx,
      createQuoteAtaIx,
      transferSolIx,
      syncNativeIx,
      buyIx,
    ]),
  );

  // ── Step 3: Migrate ───────────────────────────────────────────────────────
  const migrateLaunchIxBase = initializer.createMigrateLaunchInstruction({
    config: initializerConfig,
    launch,
    launchAuthority,
    baseMint: baseMint.address,
    quoteMint: WSOL_MINT,
    baseVault: baseVault.address,
    quoteVault: quoteVault.address,
    migratorProgram: cpmmMigrator.CPMM_MIGRATOR_PROGRAM_ID,
    payer,
    baseTokenProgram: TOKEN_PROGRAM_ADDRESS,
    quoteTokenProgram: TOKEN_PROGRAM_ADDRESS,
    systemProgram: SYSTEM_PROGRAM_ADDRESS,
    rent: SYSVAR_RENT_ADDRESS,
  });

  const migrateLaunchIx = {
    ...migrateLaunchIxBase,
    accounts: [
      ...(migrateLaunchIxBase.accounts ?? []),
      { address: cpmmMigratorState, role: AccountRole.WRITABLE },
      { address: cpmmConfig, role: AccountRole.READONLY },
      { address: pool, role: AccountRole.WRITABLE },
      { address: poolAuthority, role: AccountRole.READONLY },
      { address: poolVault0, role: AccountRole.WRITABLE },
      { address: poolVault1, role: AccountRole.WRITABLE },
      { address: protocolPosition, role: AccountRole.WRITABLE },
      { address: launchLpPosition, role: AccountRole.WRITABLE },
      { address: cpmm.CPMM_PROGRAM_ID, role: AccountRole.READONLY },
      { address: migrationAuthority, role: AccountRole.READONLY },
      { address: adminBaseAta, role: AccountRole.WRITABLE },
      { address: adminQuoteAta, role: AccountRole.WRITABLE },
    ],
  };

  console.log('Migrate:', await send([migrateLaunchIx]));
}

main();
```
