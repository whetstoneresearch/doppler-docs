---
description: Easily launch new assets on Solana with Doppler
---

# Launch

```typescript
import {
  createKeyPairSignerFromBytes,
  createSolanaRpc,
  createSolanaRpcSubscriptions,
  generateKeyPairSigner,
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

const WSOL_MINT: Address =
  'So11111111111111111111111111111111111111112' as Address;

async function getSolPriceUsd(): Promise<number> {
  const response = await fetch(
    'https://api.coingecko.com/api/v3/simple/price?ids=solana&vs_currencies=usd',
  );
  const data = await response.json();
  return data.solana.usd;
}

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
  const MIN_RAISE_QUOTE = 50n * 1_000_000_000n;
  const SWAP_FEE_BPS = 200;

  const { start } = cpmm.marketCapToCurveParams({
    startMarketCapUSD: 100_000,
    endMarketCapUSD: 10_000_000,
    baseTotalSupply: BASE_TOTAL_SUPPLY,
    baseForCurve: BASE_TOTAL_SUPPLY,
    baseDecimals: BASE_DECIMALS,
    quoteDecimals: 9,
    numerairePriceUSD: await getSolPriceUsd(),
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

  const migrationAccounts =
    await cpmmMigrator.buildCpmmMigrationRemainingAccounts({
      launch,
      baseMint: baseMint.address,
      quoteMint: WSOL_MINT,
      launchAuthority,
      adminBaseAta,
      adminQuoteAta,
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
        quoteMint: WSOL_MINT,
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

  const signature = await send([initializeLaunchIx]);
  const launchAccount = await initializer.fetchLaunch(rpc, launch, {
    programId: deployment.initializerProgram,
  });

  console.log('Launch created:', launch);
  console.log('Base mint:', baseMint.address);
  console.log('Transaction:', signature);
  console.log(
    'Phase:',
    launchAccount && initializer.phaseLabel(launchAccount.phase),
  );
}

main();
```
