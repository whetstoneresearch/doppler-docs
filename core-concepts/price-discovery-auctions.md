---
description: Learn about Doppler's price discovery auctions
icon: shop-24
---

# Price discovery auctions

## Price Discovery Auctions

Doppler uses **on-chain price discovery auctions** to determine fair market prices for a new token before migrating liquidity into a live AMM. Auctions let real users compete for tokens based on supply and demand rather than fixed pricing, resulting in transparent and efficient price formation.

***

### What Is a Price Discovery Auction

A price discovery auction is a market mechanism where participants place bids to acquire tokens. The final price emerges from buyer demand, supply constraints, and bidding behavior - revealing a market-clearing price that reflects true value in an environment with asymmetric information.&#x20;

In Doppler, this process allows token issuers to launch without having to attempt and price their own assets, letting the market determine the value while building a cohesive and liquid market.

***

### Key Auction Types Supported

* **Static Auctions** — Simplest path: price increases on a simple supply curve.
* **Multicurve Auctions** — Multiple supply curves that shape price evolution.
* **Dynamic Auctions** — Price evolves based on configuration, market demand and time.

Each kind of price discovery auction is differentiated on how price update as tokens are sold, enabling flexible mechanisms tailored to a given project's goals.

***

### How It Works (High-Level)

1. **Configure Auction Parameters**
   * Total tokens for sale
   * Curve type and slope
   * Fee routing and treasury rules
2. **Start Auction**
   * Users bid/buy along the curve
   * On-chain events record demand and fill
3. **Determine Clearing Price**
   * The auction clears when supply is exhausted or duration ends
   * The resulting price reflects aggregated demand
4. **Finalize & Migrate**
   * Auction proceeds are used to seed liquidity
   * Liquidity is migrated into a target AMM

This yields a **market-established price** and a live trading pool without manual pricing.

***

### Why Auctions Matter

* 📈 **Fair pricing** — Market forces set the initial price.
* 🔄 **Transparent mechanics** — On-chain bidding reveals demand.
* 💧 **Efficient liquidity** — Auction proceeds form the foundation of the trading pool.
* 📊 **Demand-driven launch** — Prevents arbitrary initial pricing.

By embedding price discovery into token initialization, Doppler aligns incentives between issuers and early participants, resulting in the best possible outcomes.
