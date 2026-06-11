---
description: Create a Solana launch, buy from the curve, and migrate to CPMM
---

# Launch, buy, and migrate

```typescript
import {
  createKeyPairSignerFromBytes,
  createSolanaRpc,
  createSolanaRpcSubscriptions,
  generateKeyPairSigner,
  getBase64EncodedWireTransaction,
  pipe,
  createTransactionMessage,
  setTransactionMessageFeePayerSigner,
  setTransactionMessageLifetimeUsingBlockhash,
  appendTransactionMessageInstructions,
  signTransactionMessageWithSigners,
  sendAndConfirmTransactionFactory,
  getSignatureFromTransaction,
  type Address,
  type Instruction,
} from '@solana/kit';
import {
  TOKEN_PROGRAM_ADDRESS,
  findAssociatedTokenPda,
  getCreateAssociatedTokenIdempotentInstruction,
} from '@solana-program/token';
import { SYSTEM_PROGRAM_ADDRESS } from '@solana-program/system';
import { SYSVAR_RENT_ADDRESS } from '@solana/sysvars';
import {
  cpmm,
  cpmmMigrator,
  initializer,
  deriveSolanaCpmmDeployment,
  DOPPLER_SOLANA_DEVNET_PROGRAM_ADDRESSES,
} from '@whetstone-research/doppler-sdk/solana';

const keypairJson = process.env.SOLANA_KEYPAIR;
const rpcUrl = process.env.SOLANA_RPC_URL ?? 'https://api.devnet.solana.com';
const wsUrl = process.env.SOLANA_WS_URL ?? 'wss://api.devnet.solana.com';

if (!keypairJson) {
  throw new Error('SOLANA_KEYPAIR must be set (JSON array of 64 bytes)');
}

const USDC_MINT: Address =
  '4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU' as Address;

async function main() {
  const payer = await createKeyPairSignerFromBytes(
    new Uint8Array(JSON.parse(keypairJson as string)),
  );
  const rpc = createSolanaRpc(rpcUrl);
  const rpcSubscriptions = createSolanaRpcSubscriptions(wsUrl);
  const deployment = await deriveSolanaCpmmDeployment(
    DOPPLER_SOLANA_DEVNET_PROGRAM_ADDRESSES,
  );
  const sendAndConfirmTransaction = sendAndConfirmTransactionFactory({
    rpc,
    rpcSubscriptions,
  });

  async function send(instructions: Instruction[]) {
    const { value: latestBlockhash } = await rpc.getLatestBlockhash().send();
    const message = pipe(
      createTransactionMessage({ version: 0 }),
      (tx) => setTransactionMessageFeePayerSigner(payer, tx),
      (tx) => setTransactionMessageLifetimeUsingBlockhash(latestBlockhash, tx),
      (tx) => appendTransactionMessageInstructions(instructions, tx),
    );
    const signedTransaction = await signTransactionMessageWithSigners(message);
    await sendAndConfirmTransaction(
      signedTransaction as Parameters<typeof sendAndConfirmTransaction>[0],
      { commitment: 'confirmed' },
    );
    return getSignatureFromTransaction(signedTransaction);
  }

  const BASE_DECIMALS = 6;
  const BASE_TOTAL_SUPPLY = 1_000_000_000n * 10n ** BigInt(BASE_DECIMALS);
  const MIN_RAISE_QUOTE = 100_000n;
  const BUY_AMOUNT_IN = 200_000n;
  const SWAP_FEE_BPS = 200;

  const { start } = cpmm.marketCapToCurveParams({
    startMarketCapUSD: 100_000,
    endMarketCapUSD: 10_000_000,
    baseTotalSupply: BASE_TOTAL_SUPPLY,
    baseForCurve: BASE_TOTAL_SUPPLY,
    baseDecimals: BASE_DECIMALS,
    quoteDecimals: 6,
    numerairePriceUSD: 1,
  });

  const baseMint = await generateKeyPairSigner();
  const baseVault = await generateKeyPairSigner();
  const quoteVault = await generateKeyPairSigner();

  const namespace = payer.address;
  const launchId = initializer.launchIdFromU64(BigInt(Date.now()));
  const [launch] = await initializer.getLaunchAddress(
    namespace,
    launchId,
    deployment.initializerProgram,
  );
  const [launchAuthority] = await initializer.getLaunchAuthorityAddress(
    launch,
    deployment.initializerProgram,
  );
  const [launchFeeState] = await initializer.getLaunchFeeStateAddress(
    launch,
    deployment.initializerProgram,
  );
  const [payerBaseAta] = await findAssociatedTokenPda({
    owner: payer.address,
    mint: baseMint.address,
    tokenProgram: TOKEN_PROGRAM_ADDRESS,
  });
  const [payerQuoteAta] = await findAssociatedTokenPda({
    owner: payer.address,
    mint: USDC_MINT,
    tokenProgram: TOKEN_PROGRAM_ADDRESS,
  });

  const migrationAccounts =
    await cpmmMigrator.buildCpmmMigrationRemainingAccounts({
      launch,
      baseMint: baseMint.address,
      quoteMint: USDC_MINT,
      launchAuthority,
      adminBaseAta: payerBaseAta,
      adminQuoteAta: payerQuoteAta,
      recipientAtas: [],
      cpmmProgram: deployment.cpmmProgram,
      cpmmMigratorProgram: deployment.cpmmMigratorProgram,
    });

  const migratorInitPayload = cpmmMigrator.encodeRegisterLaunchPayload({
    cpmmConfig: migrationAccounts.cpmmConfig,
    initialSwapFeeBps: SWAP_FEE_BPS,
    initialFeeSplitBps: 10_000,
    recipients: [],
    minRaiseQuote: MIN_RAISE_QUOTE,
    minMigrationPriceQ64Opt: null,
    migratedPoolHookConfig: null,
  });
  const migratorMigratePayload = cpmmMigrator.encodeMigratePayload({
    baseForDistribution: 0n,
    baseForLiquidity: 0n,
  });

  const initializeLaunchIx =
    await initializer.createInitializeLaunchInstruction(
      {
        config: deployment.initializerConfig,
        launch,
        launchAuthority,
        baseMint,
        quoteMint: USDC_MINT,
        baseVault,
        quoteVault,
        launchFeeState,
        payer,
        authority: payer,
        hookProgram: deployment.cpmmHookProgram,
        migratorProgram: deployment.cpmmMigratorProgram,
        cpmmConfig: migrationAccounts.cpmmConfig,
        baseTokenProgram: TOKEN_PROGRAM_ADDRESS,
        quoteTokenProgram: TOKEN_PROGRAM_ADDRESS,
        systemProgram: SYSTEM_PROGRAM_ADDRESS,
        rent: SYSVAR_RENT_ADDRESS,
      },
      {
        namespace,
        launchId,
        baseDecimals: BASE_DECIMALS,
        baseTotalSupply: BASE_TOTAL_SUPPLY,
        baseForDistribution: 0n,
        baseForLiquidity: 0n,
        curveVirtualBase: start.curveVirtualBase,
        curveVirtualQuote: start.curveVirtualQuote,
        swapFeeBps: SWAP_FEE_BPS,
        curveKind: initializer.CURVE_KIND_XYK,
        curveParams: new Uint8Array([initializer.CURVE_PARAMS_FORMAT_XYK_V0]),
        allowBuy: true,
        allowSell: true,
        hookFlags: initializer.HF_BEFORE_SWAP,
        hookPayload: new Uint8Array(),
        migratorInitPayload,
        migratorMigratePayload,
        hookRemainingAccountsHash: initializer.EMPTY_REMAINING_ACCOUNTS_HASH,
        migratorInitRemainingAccountsHash:
          initializer.computeRemainingAccountsHash([
            migrationAccounts.cpmmMigrationState,
            migrationAccounts.cpmmConfig,
          ]),
        migratorRemainingAccountsHash: migrationAccounts.hash,
        feeBeneficiaries: [{ wallet: payer.address, shareBps: 10_000 }],
        metadataName: '',
        metadataSymbol: '',
        metadataUri: '',
      },
      deployment.initializerProgram,
    );

  console.log('Launch transaction:', await send([initializeLaunchIx]));
  console.log('Launch:', launch);
  console.log('Base mint:', baseMint.address);

  const previewIx = initializer.createPreviewSwapExactInInstruction(
    {
      launch,
      launchFeeState,
      baseVault: baseVault.address,
      quoteVault: quoteVault.address,
      hookProgram: deployment.cpmmHookProgram,
    },
    {
      amountIn: BUY_AMOUNT_IN,
      tradeDirection: initializer.TRADE_DIRECTION_BUY,
    },
    deployment.initializerProgram,
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
  const returnData = simulateResult.returnData?.data;
  if (returnData) {
    const bytes = Uint8Array.from(atob(returnData[0]), (c) => c.charCodeAt(0));
    const preview = initializer.decodePreviewSwapExactInResult(bytes);
    console.log('Preview amount out:', preview.amountOut.toString());
  }

  const createBaseAtaIx = getCreateAssociatedTokenIdempotentInstruction({
    payer,
    ata: payerBaseAta,
    owner: payer.address,
    mint: baseMint.address,
  });
  const createQuoteAtaIx = getCreateAssociatedTokenIdempotentInstruction({
    payer,
    ata: payerQuoteAta,
    owner: payer.address,
    mint: USDC_MINT,
  });
  const buyIx = initializer.createCurveSwapExactInInstruction(
    {
      config: deployment.initializerConfig,
      launch,
      launchAuthority,
      baseVault: baseVault.address,
      quoteVault: quoteVault.address,
      launchFeeState,
      userBaseAccount: payerBaseAta,
      userQuoteAccount: payerQuoteAta,
      baseMint: baseMint.address,
      quoteMint: USDC_MINT,
      user: payer,
      hookProgram: deployment.cpmmHookProgram,
      baseTokenProgram: TOKEN_PROGRAM_ADDRESS,
      quoteTokenProgram: TOKEN_PROGRAM_ADDRESS,
    },
    {
      amountIn: BUY_AMOUNT_IN,
      minAmountOut: 1n,
      tradeDirection: initializer.TRADE_DIRECTION_BUY,
    },
    deployment.initializerProgram,
  );
  console.log(
    'Buy transaction:',
    await send([createBaseAtaIx, createQuoteAtaIx, buyIx]),
  );

  const migrateLaunchIxBase = initializer.createMigrateLaunchInstruction(
    {
      config: deployment.initializerConfig,
      launch,
      launchAuthority,
      baseMint: baseMint.address,
      quoteMint: USDC_MINT,
      baseVault: baseVault.address,
      quoteVault: quoteVault.address,
      launchFeeState,
      migratorProgram: deployment.cpmmMigratorProgram,
      payer,
      baseTokenProgram: TOKEN_PROGRAM_ADDRESS,
      quoteTokenProgram: TOKEN_PROGRAM_ADDRESS,
      systemProgram: SYSTEM_PROGRAM_ADDRESS,
      rent: SYSVAR_RENT_ADDRESS,
    },
    deployment.initializerProgram,
  );
  const migrateLaunchIx = {
    ...migrateLaunchIxBase,
    accounts: [
      ...(migrateLaunchIxBase.accounts ?? []),
      ...migrationAccounts.metas,
    ],
  };

  console.log('Migration transaction:', await send([migrateLaunchIx]));
}

main();
```
