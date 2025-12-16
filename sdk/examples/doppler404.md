---
description: Create hybrid fungible and non-fungible tokens
---

# Doppler404

### About

Doppler404 enables token creators to issue assets with unified liquid markets by bridging the gap between non-fungible tokens (ERC-721) and fungible tokens (ERC-20). This provides automated price discovery, sniping mitigation, a unified liquid market, and an improved trading UX over traditional NFTs - in addition to expanding the utility of fungible tokens for a variety of key use cases.

**How it works**

Creators and users can define a custom quantity of fungible tokens that result in NFT ownership as inputs into the Doppler protocol contracts. For example, holding 100 of a specific fungible token, eg. $ABC, can result in holding 1 NFT from an art collection called ABCart. These assets can be auctioned using either Doppler's static or dynamic auctions, and migrate the liquidity generated to any of the supported AMMs, with or without onchain governance, just like other Doppler token configurations.&#x20;

#### DN404 & ERC-7631 modifications

Doppler404 builds on top of the foundation of “DN404”, aka [ERC-7631 for “Dual Nature Token Pairs"](https://ethereum-magicians.org/t/erc-7631-dual-nature-token-pair/18796). With a few modifications, Doppler extends the [DN404](https://github.com/Vectorized/dn404) token standard allowing users to freeze their token balances, ensuring users don't accidentally trade away NFTs that they wish to hold.

### Example usage&#x20;

```typescript
const dynamicBuilder = new DynamicAuctionBuilder()
  .tokenConfig({
    type: 'doppler404' as const,
    name: data.tokenName,
    symbol: data.tokenSymbol,
    baseURI: data.baseURI || `https://metadata.example.com/${data.symbol}/`,
  })
  .saleConfig({ initialSupply, numTokensToSell })
  .poolConfig({ fee: 3000, tickSpacing: 8 })
  .auctionByTicks({
    startTick, endTick,
    minProceeds: parseEther("100"),
    maxProceeds: parseEther("600"),
    durationDays: 7,
    epochLength: 3600,
  })
  .withMigration({
    type: 'uniswapV4', fee: 3000, tickSpacing: 60,
    streamableFees: {
      lockDuration: 60 * 60 * 24 * 30,
      beneficiaries: []
    }
  })
  .withGovernance({ type: 'noOp' })
  .withUserAddress(account.address)
  .withIntegrator()
  .withTime({ blockTimestamp: Number(adjustedTimestamp) })
  .build();
```

### Demo application

The Doppler Demo App supports Doppler404 and is the easiest way to get started.

<figure><img src="../../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

View it on GitHub: [https://github.com/whetstoneresearch/doppler-demo-app](https://github.com/whetstoneresearch/doppler-demo-app)&#x20;
