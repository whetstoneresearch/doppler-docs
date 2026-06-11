---
description: Easily swap Doppler created assets on Solana
---

# Swap

```typescript
import {
  address,
  createKeyPairSignerFromBytes,
  createSolanaRpc,
  createSolanaRpcSubscriptions,
  pipe,
  createTransactionMessage,
  setTransactionMessageFeePayerSigner,
  setTransactionMessageLifetimeUsingBlockhash,
  appendTransactionMessageInstructions,
  signTransactionMessageWithSigners,
  sendAndConfirmTransactionFactory,
  getSignatureFromTransaction,
  type Address,
} from '@solana/kit';
import {
  TOKEN_PROGRAM_ADDRESS,
  findAssociatedTokenPda,
  getCreateAssociatedTokenIdempotentInstruction,
} from '@solana-program/token';
import {
  cpmm,
  deriveSolanaCpmmDeployment,
  DOPPLER_SOLANA_DEVNET_PROGRAM_ADDRESSES,
} from '@whetstone-research/doppler-sdk/solana';

const keypairJson = process.env.SOLANA_KEYPAIR;
const rpcUrl = process.env.SOLANA_RPC_URL ?? 'https://api.devnet.solana.com';
const wsUrl = process.env.SOLANA_WS_URL ?? 'wss://api.devnet.solana.com';

if (!keypairJson) {
  throw new Error('SOLANA_KEYPAIR must be set (JSON array of 64 bytes)');
}
if (!process.env.MINT_0 || !process.env.MINT_1) {
  throw new Error('MINT_0 and MINT_1 must be set');
}

const MINT_0: Address = address(process.env.MINT_0);
const MINT_1: Address = address(process.env.MINT_1);

async function main() {
  const payer = await createKeyPairSignerFromBytes(
    new Uint8Array(JSON.parse(keypairJson as string)),
  );
  const rpc = createSolanaRpc(rpcUrl);
  const rpcSubscriptions = createSolanaRpcSubscriptions(wsUrl);
  const deployment = await deriveSolanaCpmmDeployment(
    DOPPLER_SOLANA_DEVNET_PROGRAM_ADDRESSES,
  );

  const poolResult = await cpmm.getPoolByMints(rpc, MINT_0, MINT_1, {
    programId: deployment.cpmmProgram,
  });
  if (!poolResult) {
    throw new Error(`No pool found for ${MINT_0} / ${MINT_1}`);
  }

  const { address: poolAddress, account: pool } = poolResult;
  const tradeDirection = (process.env.SWAP_DIRECTION === '1' ? 1 : 0) as
    | 0
    | 1;
  const amountIn = BigInt(process.env.AMOUNT_IN ?? '1000000');
  const quote = cpmm.getSwapQuote(pool, amountIn, tradeDirection);
  const minAmountOut = (quote.amountOut * 9950n) / 10000n;

  const [userToken0] = await findAssociatedTokenPda({
    owner: payer.address,
    mint: pool.token0Mint,
    tokenProgram: TOKEN_PROGRAM_ADDRESS,
  });
  const [userToken1] = await findAssociatedTokenPda({
    owner: payer.address,
    mint: pool.token1Mint,
    tokenProgram: TOKEN_PROGRAM_ADDRESS,
  });

  const createUserToken0Ix = getCreateAssociatedTokenIdempotentInstruction({
    payer,
    ata: userToken0,
    owner: payer.address,
    mint: pool.token0Mint,
  });
  const createUserToken1Ix = getCreateAssociatedTokenIdempotentInstruction({
    payer,
    ata: userToken1,
    owner: payer.address,
    mint: pool.token1Mint,
  });
  const swapIx = cpmm.createSwapInstruction({
    config: deployment.cpmmConfig,
    pool: poolAddress,
    authority: pool.authority,
    vault0: pool.vault0,
    vault1: pool.vault1,
    token0Mint: pool.token0Mint,
    token1Mint: pool.token1Mint,
    userToken0,
    userToken1,
    user: payer,
    amountIn,
    minAmountOut,
    tradeDirection,
    programId: deployment.cpmmProgram,
  });

  const { value: latestBlockhash } = await rpc.getLatestBlockhash().send();
  const transactionMessage = pipe(
    createTransactionMessage({ version: 0 }),
    (tx) => setTransactionMessageFeePayerSigner(payer, tx),
    (tx) => setTransactionMessageLifetimeUsingBlockhash(latestBlockhash, tx),
    (tx) =>
      appendTransactionMessageInstructions(
        [createUserToken0Ix, createUserToken1Ix, swapIx],
        tx,
      ),
  );

  const signedTransaction =
    await signTransactionMessageWithSigners(transactionMessage);
  const sendAndConfirmTransaction = sendAndConfirmTransactionFactory({
    rpc,
    rpcSubscriptions,
  });

  await sendAndConfirmTransaction(
    signedTransaction as Parameters<typeof sendAndConfirmTransaction>[0],
    { commitment: 'confirmed' },
  );

  console.log('Swap confirmed:', getSignatureFromTransaction(signedTransaction));
}

main();
```
