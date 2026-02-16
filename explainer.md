---
icon: lightbulb
---

# Explainer

[Doppler](https://doppler.lol) is an onchain protocol for launching tokens through various [price discovery auctions](https://aada.ms/pdfs/pda.pdf).

Teams use Doppler to create new tokens and determine their prices via onchain public markets. In addition to basic creation of the digital assets, Doppler can handle various downstream functionality, such as vesting, inflation, governance, multiparty trading fees, buybacks, and more.

Doppler's [smart contracts](https://github.com/whetstoneresearch/doppler) can be interacted with directly or through an open source [SDK](https://github.com/whetstoneresearch/doppler-sdk) which abstracts away all relevant smart contract complexity. Data including market cap, volume, holder counts, and more is available through a publicly hosted [indexing API](indexer/api-usage.md).  All of the smart contracts have been through [security review &/or audited](resources/security-and-bug-bounties.md).

### Price discovery auctions

Doppler currently supports three different kinds of price discovery auctions, each designed to be utilized in different contexts, such as available network compatibility or desired market structure.

* Static auctions - allow specifying a single supply or price curve.
* Multicurve auctions - allow specifying multiple supply or price curves.
* Dynamic auctions - allow specifying ranges for dynamically adjusting supply or price curves.

### Liquidity migration

After a price discovery auction completes, the liquidity proceeds generated in the price discovery auction can be migrated into a different DeFi protocol or automated market maker, automtaing end to end liquidity pool creation. Integrations with Uniswap support fee rehypothecation, enabling fees to be programmatically redirected, e.g., to grow liquidity, perform buybacks, or consolidate into one side of the market. Pools can be enabled with Doppler Hooks, easily enabling custom fee logic and hook-based extensions. Custom modules can also be implemented to support migration to arbitrary AMMs or protocols, including non-Uniswap deployments.

### Fees

Every token can be created with a unique set of beneficiary addresses that earn earn fees on every swap in pools created by Doppler, forever. Fees can also optionally be configured to decay from the time they're created in efforts to mitigate the inecntives of automated purchasing, for example, starting at 80% and decreasing linearly down to 1% over the first N seconds. The Doppler Protocol fee is set to 5% of whatever the token's fees are configured to. For example, if a token is configured with 1% fees, the protocol earns 0.05%.

### Governance & Treasury

Using OpenZeppelin's [Governor](https://docs.openzeppelin.com/contracts/4.x/api/governance#governor), tokens can optionally be created with onchain token holder governance and treasury management. Community stakeholders can hold onto a meaningful percentage of token supply, and after vesting or unlocks, vote on what to do with it's treasury.&#x20;

### Airlock

Doppler's [Airlock](https://github.com/whetstoneresearch/doppler/blob/main/src/Airlock.sol) smart contract provides a unified interface for interacting with the protocol. Users pass their desired parameters into the Airlock, which then manages all interactions with downstream contracts - including token factories, hook initializers, governance factories, and any other contracts configured for a given launch. The [Doppler SDK](https://github.com/whetstoneresearch/doppler-sdk) makes interactions with the Doppler SDK seamless and intuitive, abstracting away smart contract complexity.

### Design principles

* **Capital efficiency** - Highly optimized auctions help tokens find their fair market price.
* **MEV protecting** - Configure custom markets and launches designed to mitigate sniping & MEV. &#x20;
* **Composability -** Doppler is fully onchain and composable with other programs.
* **Optionality -** turn on and off different modules, eg. use governance, or opt out of it entirely.
