# Overview

### Context

Doppler was initially released with two SDKs — one for the v3 version of the protocol that facilitates static bonding curves, and another for the v4 version of the protocol, facilitating dynamic bonding curves with dutch auctions. This decision was inspired by Uniswap having different SDKs and versioning for different versions of the Uniswap protocol.&#x20;

Doppler has decided in an effort to optimize developer experience and ease of switching between different versions of the protocol to _consolidate the two SDKs_ into a single package and interface. Additionally we have chosen to implement a friendly and intuitive builder pattern, making it even easier to configure auctions, governance, vesting, liquidity migration, and quotes/swapping.

## Status

The original v3/v4 Doppler SDKs are officially deprecated and no longer supported. Teams using these SDKs are encouraged to migrate as soon as possible using this migration guide. The legacy SDKs are hosted at https://github.com/whetstoneresearch/doppler-sdk-legacy. The new, officially supported SDK (previously referred to as the "alpha" SDK) is now hosted at https://github.com/whetstoneresearch/doppler-sdk and released in Beta. This is currently used in the Doppler Application and various third party applications, making it the official recommendation. 
