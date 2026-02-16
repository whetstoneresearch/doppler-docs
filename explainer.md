---
icon: lightbulb
---

# Explainer

[Doppler](https://doppler.lol) is an onchain protocol for launching tokens through various [price discovery auctions](https://aada.ms/pdfs/pda.pdf).

Teams use Doppler to create new tokens and determine their prices via onchain public markets. In addition to basic creation of the digital assets, Doppler can handle various downstream functionality, such as vesting, inflation, governance, multiparty trading fees, buybacks, and more.

Doppler's [smart contracts](https://github.com/whetstoneresearch/doppler) can be interacted with directly or through an open source [SDK](https://github.com/whetstoneresearch/doppler-sdk) which abstracts away all relevant smart contract complexity. Data including market cap, volume, holder counts, and more is available through a publicly hosted [indexing API](indexer/api-usage.md).  

All of the smart contracts have been through [security review &/or audited](resources/security-and-bug-bounties.md).

## Key concepts

### Doppler Airlock

Doppler's [Airlock](https://github.com/whetstoneresearch/doppler/blob/main/src/Airlock.sol) smart contract provides a unified interface for interacting with the protocol. Users pass their desired parameters into the Airlock, which then manages all interactions with downstream contracts - including token factories, hook initializers, governance factories, and any other contracts configured for a given launch. 

### Price discovery auctions

Doppler currently supports three different kinds of price discovery auctions, each designed to be utilized in different contexts, such as available network compatibility or desired market structure.

* Static auctions - allow specifying a single supply or price curve.
* Multicurve auctions - allow specifying multiple supply or price curves.
* Dynamic auctions - allow specifying ranges for dynamically adjusting supply or price curves.

### Liquidity migration

After a price discovery auction completes, the liquidity proceeds generated in the price discovery auction can be migrated into a different DeFi protocol or automated market maker, automtaing end to end liquidity pool creation. Doppler supports fee rehypothecation, enabling fees to be programmatically redirected, e.g., to grow liquidity, perform buybacks, or consolidate into one side of the market. 

### Doppler Hooks

Use custom callback functions on initialization, each swap, or token graduation, aka liquidity migration at a specified price or proceed target. This can enable custom logic or downstream triggers within external contracts. See the [Doppler Hooks](/resources/doppler-hooks.md) section for more information.

### Fees

Every token can be created with a unique set of beneficiary addresses that earn earn fees on every swap in pools created by Doppler, forever. Fees can also optionally be configured to decay from the time they're created in efforts to mitigate the inecntives of automated purchasing, for example, starting at 80% and decreasing linearly down to 1% over the first N seconds. 

#### Protocol fee 

The Doppler Protocol fee is set to 5% of whatever the newly created token's fees are configured to. For example, if a token is configured with 1% fees, the protocol earns 0.05%.. or a token configured with 0% fees earns the protocol 0%. 

### Governance & Treasury

Using OpenZeppelin's [Governor](https://docs.openzeppelin.com/contracts/4.x/api/governance#governor), tokens can optionally be created with onchain token holder governance and treasury management. Community stakeholders can hold onto a meaningful percentage of token supply, and after vesting or unlocks, vote on what to do with it's treasury.

### Design principles

* **Capital efficiency** - Highly optimized auctions help tokens find their fair market price.
* **MEV protecting** - Configure custom markets and launches designed to mitigate sniping & MEV.
* **Composability -** Doppler is fully onchain and composable with other programs.
* **Optionality -** turn on and off different modules, eg. use governance, or opt out of it entirely.

### Custom modules

Smart contracts called "modules" can be added to Doppler and used alongside the Airlock and rest of the protocol. This provides necessary customization or differentiation while still utilzing the rest of Doppler's end to end battle tested infrastructure. For example, teams could implement a custom module to migrate liquidity to a non-Uniswap AMM, or into a different DeFi protocol entirely, or teams choose to implement custom modules for specific compliance considerations or requirements. 

### Usage & Integrations

The [Doppler Protocol](https://github.com/whetstoneresearch/doppler)'s smart contracts can be interacted with directly. The [Doppler TypeScript SDK](https://github.com/whetstoneresearch/doppler-sdk) enables seamless application integrations, abstracting away underlying smart contract complexity. The [Doppler Application](https://app.doppler.lol) provides a self-service interface for teams that want off-the-shelf customization without writing code.
