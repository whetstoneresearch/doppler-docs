# Overview

### Context

Doppler was initially released with two SDKs — one for the v3 version of the protocol that facilitates static bonding curves, and another for the v4 version of the protocol, facilitating dynamic bonding curves with dutch auctions. This decision was inspired by Uniswap having different SDKs and versioning for different versions of the Uniswap protocol.&#x20;

Doppler has decided in an effort to optimize developer experience and ease of switching between different versions of the protocol to _consolidate the two SDKs_ into a single package and interface. Additionally we have chosen to implement a friendly and intuitive builder pattern, making it even easier to configure auctions, governance, vesting, liquidity migration, and quotes/swapping.&#x20;

_**It is now available in alpha.**_ Please use with caution until it stabilizes in production and moves into beta.

### Deprecation notice for the v3/v4 SDKs

Once this SDK has reached production maturity the Whetstone Research team plans to deprecate and archive the previous versions of the Doppler v3 and v4 SDKs contained in the monorepo at [https://github.com/whetstoneresearch/doppler-sdk](https://github.com/whetstoneresearch/doppler-sdk). At that time, it will be encouraged that teams using old versions of the SDK migrate to this new, consolidated SDK. See the [migration guide](sdk-migration-guide.md).

Notably, new feature additions will _only_ be added to this SDK moving forward.&#x20;

Timeline: end of Q3 or beginning of Q4, 2025 (8+ weeks) depending on stability and feedback.
