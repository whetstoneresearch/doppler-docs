---
description: Learn about creating tokens using the Doppler Protocol
icon: folder-plus
---

# Creation & initialization

## Creation & Initialization

Doppler lets you create new digital assets and initialize liquidity markets in a single, configurable flow.

Using the SDK or Doppler App, you can:

* Deploy a new token
* Configure its supply distribution
* Define price discovery mechanics
* Route fees and treasury allocations
* Initialize liquidity into an AMM or custom market

***

### What You Can Create

Doppler supports multiple asset configurations:

* **Standard ERC-20 tokens**
* Tokens with **custom supply curves**
* Tokens with **vesting or inflation schedules**
* Assets with **programmable fee routing**
* Governance-ready or treasury-aware tokens

All token economics are defined at creation.

***

### Price Discovery Options

Instead of seeding a traditional LP directly, Doppler supports configurable price discovery mechanisms:

* Static price curves
* Multicurve distributions
* Dynamic / time-based curves

This allows liquidity to form through transparent on-chain trading before migration to a live AMM.

***

### Initializing Liquidity

After price discovery completes, liquidity proceeds can be:

* Migrated into a Uniswap-style AMM
* Consolidated into one side of the market
* Used for automated buybacks
* Directed to treasury or custom fee destinations

Liquidity initialization is programmable and defined at creation.

***

### High-Level Flow

1. Configure token + economics
2. Deploy via SDK or App
3. Run price discovery auction
4. Finalize and migrate liquidity
5. Market becomes live on target AMM

***

### Why Use Doppler for Creation?

* No custom smart contract development required
* On-chain, transparent distribution
* Configurable fee and treasury mechanics
* Clean migration into standard AMMs

Doppler turns token creation and liquidity initialization into a single, programmable system.
