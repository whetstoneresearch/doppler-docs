---
icon: pen-field
---

# Contract addresses

Here are the networks that Doppler is officially deployed to:

* Mainnets: Base, Unichain, Ink&#x20;
* Testnets: Unichain Sepolia, Base Sepolia, Ink Sepolia, Monad Testnet

Note that contracts addresses are not the same across the different networks.

{% hint style="danger" %}
Doppler v4 with dynamic bonding curves is in beta. Use at your own risk and reach out to the Whetstone Research team about implementation and parameterization best practices. :rotating\_light: If there are contracts not reflected here but claiming to be instances of Doppler, they are not considered canonical.
{% endhint %}

## Mainnet Deployments

### Base (8453)

| Contract             | Address                                                                                                               |
| -------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Airlock              | [0x660eAaEdEBc968f8f3694354FA8EC0b4c5Ba8D12](https://basescan.org/address/0x660eAaEdEBc968f8f3694354FA8EC0b4c5Ba8D12) |
| TokenFactory         | [0xFAafdE6a5b658684cC5eb0C5c2c755B00A246F45](https://basescan.org/address/0xFAafdE6a5b658684cC5eb0C5c2c755B00A246F45) |
| UniswapV3Initializer | [0xaA47D2977d622DBdFD33eeF6a8276727c52EB4e5](https://basescan.org/address/0xaA47D2977d622DBdFD33eeF6a8276727c52EB4e5) |
| UniswapV4Initializer | [0x77EbfBAE15AD200758E9E2E61597c0B07d731254](https://basescan.org/address/0x77EbfBAE15AD200758E9E2E61597c0B07d731254) |
| DopplerDeployer      | [0x5CadB034267751a364dDD4d321C99E07A307f915](https://basescan.org/address/0x5CadB034267751a364dDD4d321C99E07A307f915) |
| GovernanceFactory    | [0xb4deE32EB70A5E55f3D2d861F49Fb3D79f7a14d9](https://basescan.org/address/0xb4deE32EB70A5E55f3D2d861F49Fb3D79f7a14d9) |
| UniswapV2Migrator    | [0x5F3bA43D44375286296Cb85F1EA2EBfa25dde731](https://basescan.org/address/0x5F3bA43D44375286296Cb85F1EA2EBfa25dde731) |
| Bundler              | [0x136191B46478cAB023cbC01a36160C4Aad81677a](https://basescan.org/address/0x136191B46478cAB023cbC01a36160C4Aad81677a) |
| Lens                 | [0x43d0D97EC9241A8F05A264f94B82A1d2E600f2B3](https://basescan.org/address/0x43d0D97EC9241A8F05A264f94B82A1d2E600f2B3) |

### Unichain (130)

| Contract             | Address                                                                                                                          |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Airlock              | [0x77EbfBAE15AD200758E9E2E61597c0B07d731254](https://unichain.blockscout.com/address/0x77EbfBAE15AD200758E9E2E61597c0B07d731254) |
| TokenFactory         | [0x43d0D97EC9241A8F05A264f94B82A1d2E600f2B3](https://unichain.blockscout.com/address/0x43d0D97EC9241A8F05A264f94B82A1d2E600f2B3) |
| UniswapV3Initializer | [0x9F4e56be80f08ba1A2445645EFa6d231E27b43ec](https://unichain.blockscout.com/address/0x9F4e56be80f08ba1A2445645EFa6d231E27b43ec) |
| UniswapV4Initializer | [0x2F2BAcd46d3F5c9EE052Ab392b73711dB89129DB](https://unichain.blockscout.com/address/0x2F2BAcd46d3F5c9EE052Ab392b73711dB89129DB) |
| DopplerDeployer      | [0x06FEFD02F0b6d9f57F52cfacFc113665Dfa20F0f](https://unichain.blockscout.com/address/0x06FEFD02F0b6d9f57F52cfacFc113665Dfa20F0f) |
| GovernanceFactory    | [0x99C94B9Df930E1E21a4E4a2c105dBff21bF5c5aE](https://unichain.blockscout.com/address/0x99C94B9Df930E1E21a4E4a2c105dBff21bF5c5aE) |
| UniswapV2Migrator    | [0xf6023127f6E937091D5B605680056A6D27524bad](https://unichain.blockscout.com/address/0xf6023127f6E937091D5B605680056A6D27524bad) |
| Bundler              | [0x91231cDdD8d6C86Df602070a3081478e074b97b7](https://unichain.blockscout.com/address/0x91231cDdD8d6C86Df602070a3081478e074b97b7) |
| Lens                 | [0x82Ac010C67f70BACf7655cd8948a4AD92A173CAC](https://unichain.blockscout.com/address/0x82Ac010C67f70BACf7655cd8948a4AD92A173CAC) |

### Ink (57073)

| Contract             | Address                                                                                                                          |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Airlock              | [0x660eAaEdEBc968f8f3694354FA8EC0b4c5Ba8D12](https://explorer.inkonchain.com/address/0x660eAaEdEBc968f8f3694354FA8EC0b4c5Ba8D12) |
| TokenFactory         | [0xFAafdE6a5b658684cC5eb0C5c2c755B00A246F45](https://explorer.inkonchain.com/address/0xFAafdE6a5b658684cC5eb0C5c2c755B00A246F45) |
| UniswapV3Initializer | [0xaA47D2977d622DBdFD33eeF6a8276727c52EB4e5](https://explorer.inkonchain.com/address/0xaA47D2977d622DBdFD33eeF6a8276727c52EB4e5) |
| UniswapV4Initializer | [0x014E1c0bd34f3B10546E554CB33B3293fECDD056](https://explorer.inkonchain.com/address/0x014E1c0bd34f3B10546E554CB33B3293fECDD056) |
| DopplerDeployer      | [0xa82c66b6ddEb92089015C3565E05B5c9750b2d4B](https://explorer.inkonchain.com/address/0xa82c66b6ddEb92089015C3565E05B5c9750b2d4B) |
| GovernanceFactory    | [0xb4deE32EB70A5E55f3D2d861F49Fb3D79f7a14d9](https://explorer.inkonchain.com/address/0xb4deE32EB70A5E55f3D2d861F49Fb3D79f7a14d9) |
| UniswapV2Migrator    | [0x5F3bA43D44375286296Cb85F1EA2EBfa25dde731](https://explorer.inkonchain.com/address/0x5F3bA43D44375286296Cb85F1EA2EBfa25dde731) |
| Bundler              | [0x136191B46478cAB023cbC01a36160C4Aad81677a](https://explorer.inkonchain.com/address/0x136191B46478cAB023cbC01a36160C4Aad81677a) |
| DopplerLensQuoter    | [0x8AF018e28c273826e6b2d5a99e81c8fB63729b07](https://explorer.inkonchain.com/address/0x8AF018e28c273826e6b2d5a99e81c8fB63729b07) |

## Testnet Deployments

### Unichain Sepolia (1301)

| Contract             | Address                                                                                                                                  |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Airlock              | [0x77EbfBAE15AD200758E9E2E61597c0B07d731254](https://unichain-sepolia.blockscout.com/address/0x77EbfBAE15AD200758E9E2E61597c0B07d731254) |
| TokenFactory         | [0x43d0D97EC9241A8F05A264f94B82A1d2E600f2B3](https://unichain-sepolia.blockscout.com/address/0x43d0D97EC9241A8F05A264f94B82A1d2E600f2B3) |
| UniswapV3Initializer | [0x9F4e56be80f08ba1A2445645EFa6d231E27b43ec](https://unichain-sepolia.blockscout.com/address/0x9F4e56be80f08ba1A2445645EFa6d231E27b43ec) |
| GovernanceFactory    | [0x99C94B9Df930E1E21a4E4a2c105dBff21bF5c5aE](https://unichain-sepolia.blockscout.com/address/0x99C94B9Df930E1E21a4E4a2c105dBff21bF5c5aE) |
| UniswapV2Migrator    | [0xf6023127f6E937091D5B605680056A6D27524](https://unichain-sepolia.blockscout.com/address/0xf6023127f6E937091D5B605680056A6D27524bad)    |
| Bundler              | [0x63f8C8F9beFaab2FaCD7Ece0b0242f78B920Ee90](https://unichain-sepolia.blockscout.com/address/0x63f8C8F9beFaab2FaCD7Ece0b0242f78B920Ee90) |


### Ink Sepolia

| Contract             | Address                                                                                                                                  |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Airlock              | [0x5CadB034267751a364dDD4d321C99E07A307f915](https://explorer-sepolia.inkonchain.com/address/0x5CadB034267751a364dDD4d321C99E07A307f915) |
| TokenFactory         | [0x77EbfBAE15AD200758E9E2E61597c0B07d731254](https://explorer-sepolia.inkonchain.com/address/0x77EbfBAE15AD200758E9E2E61597c0B07d731254) |
| UniswapV3Initializer | [0x43d0D97EC9241A8F05A264f94B82A1d2E600f2B3](https://explorer-sepolia.inkonchain.com/address/0x43d0D97EC9241A8F05A264f94B82A1d2E600f2B3) |
| GovernanceFactory    | [0x9F4e56be80f08ba1A2445645EFa6d231E27b43ec](https://explorer-sepolia.inkonchain.com/address/0x9F4e56be80f08ba1A2445645EFa6d231E27b43ec) |
| UniswapV2Migrator    | [0x99C94B9Df930E1E21a4E4a2c105dBff21bF5c5aE](https://explorer-sepolia.inkonchain.com/address/0x99C94B9Df930E1E21a4E4a2c105dBff21bF5c5aE) |
| Bundler              | Not deployed                                                                                                                             |

### Base Sepolia (84532)

| Contract                   | Address                                                                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Airlock                    | [0x0d2f38d807bfAd5C18e430516e10ab560D300caF](https://base-sepolia.blockscout.com/address/0x0d2f38d807bfAd5C18e430516e10ab560D300caF) |
| TokenFactory               | [0x4B0EC16Eb40318Ca5A4346f20F04A2285C19675B](https://base-sepolia.blockscout.com/address/0x4B0EC16Eb40318Ca5A4346f20F04A2285C19675B) |
| UniswapV3Initializer       | [0x1b8F12484422583FED5694469f94C7839a823980](https://base-sepolia.blockscout.com/address/0x1b8F12484422583FED5694469f94C7839a823980) |
| UniswapV4Initializer       | [0xA36715dA46Ddf4A769f3290f49AF58bF8132ED8E](https://base-sepolia.blockscout.com/address/0xA36715dA46Ddf4A769f3290f49AF58bF8132ED8E) |
| DopplerDeployer            | [0x40Bcb4dDA3BcF7dba30C5d10c31EE2791ed9ddCa](https://base-sepolia.blockscout.com/address/0x40Bcb4dDA3BcF7dba30C5d10c31EE2791ed9ddCa) |
| GovernanceFactory          | [0x65dE470Da664A5be139A5D812bE5FDa0d76CC951](https://base-sepolia.blockscout.com/address/0x65dE470Da664A5be139A5D812bE5FDa0d76CC951) |
| UniswapV2LiquidityMigrator | [0xC541FBddfEEf798E50d257495D08efe00329109A](https://base-sepolia.blockscout.com/address/0xC541FBddfEEf798E50d257495D08efe00329109A) |
| Bundler                    | [0x7E5D336A6E9e453c9f02E5102CC039E015Fd8fb8](https://base-sepolia.blockscout.com/address/0x7E5D336A6E9e453c9f02E5102CC039E015Fd8fb8) |
| Lens                       | [0x31703C016F32aC47aB71B3160b3579EcE05a5E5d](https://base-sepolia.blockscout.com/address/0x31703C016F32aC47aB71B3160b3579EcE05a5E5d) |

### World Sepolia

| Contract             | Address                                                                                                                                          |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Airlock              | [0x5F3bA43D44375286296Cb85F1EA2EBfa25dde731](https://worldchain-sepolia.explorer.alchemy.com/address/0x5F3bA43D44375286296Cb85F1EA2EBfa25dde731) |
| TokenFactory         | [0x5FBe931dc4B923A7abe4c47aD68d5bF9Eda5B76D](https://worldchain-sepolia.explorer.alchemy.com/address/0x5FBe931dc4B923A7abe4c47aD68d5bF9Eda5B76D) |
| UniswapV3Initializer | [0x9916Ec1c1E0462F6F8f7514e414F06bf001Ac82A](https://worldchain-sepolia.explorer.alchemy.com/address/0x9916Ec1c1E0462F6F8f7514e414F06bf001Ac82A) |
| GovernanceFactory    | [0x136191B46478cAB023cbC01a36160C4Aad81677a](https://worldchain-sepolia.explorer.alchemy.com/address/0x136191B46478cAB023cbC01a36160C4Aad81677a) |
| UniswapV2Migrator    | [0x8b4C7DB9121FC885689C0A50D5a1429F15AEc2a0](https://worldchain-sepolia.explorer.alchemy.com/address/0x8b4C7DB9121FC885689C0A50D5a1429F15AEc2a0) |
| Bundler              | Not deployed                                                                                                                                     |

### Monad

| Contract             | Address                                                                                                                            |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Airlock              | [0x5F3bA43D44375286296Cb85F1EA2EBfa25dde731](https://testnet.monadexplorer.com/address/0x5F3bA43D44375286296Cb85F1EA2EBfa25dde731) |
| TokenFactory         | [0x5FBe931dc4B923A7abe4c47aD68d5bF9Eda5B76D](https://testnet.monadexplorer.com/address/0x5FBe931dc4B923A7abe4c47aD68d5bF9Eda5B76D) |
| UniswapV3Initializer | [0x9916Ec1c1E0462F6F8f7514e414F06bf001Ac82A](https://testnet.monadexplorer.com/address/0x9916Ec1c1E0462F6F8f7514e414F06bf001Ac82A) |
| GovernanceFactory    | [0x136191B46478cAB023cbC01a36160C4Aad81677a](https://testnet.monadexplorer.com/address/0x136191B46478cAB023cbC01a36160C4Aad81677a) |
| UniswapV2Migrator    | [0x8b4C7DB9121FC885689C0A50D5a1429F15AEc2a0](https://testnet.monadexplorer.com/address/0x8b4C7DB9121FC885689C0A50D5a1429F15AEc2a0) |
| Bundler              | Not deployed                                                                                                                       |
