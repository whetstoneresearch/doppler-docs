# Doppler Indexer Overview

## What is the Doppler Indexer?

The Doppler Indexer is a multi-chain blockchain indexing service built for tracking the state of Doppler protocol contracts and the tokens created by those contracts. It's built to track and process real-time data from Uniswap V2, V3, and V4 protocol deployments across multiple EVM-compatible chains.

The primary goal of this indexer is to provide a reliable, fast, and queryable data source for Doppler-related analytics, front-end applications, and market analysis tools. It aggregates data on pools, tokens, swaps, liquidity, and user activity, normalizing it into a consistent schema.

## What is Ponder?

This project is built using [Ponder](https://ponder.sh/), a powerful open-source framework for building backend applications on top of blockchain data. Ponder simplifies the process of indexing event data from EVM chains by:

-   Providing a declarative way to define which contracts and events to watch.
-   Handling real-time indexing, re-orgs, and RPC interactions.
-   Generating a type-safe GraphQL API based on a defined schema.
-   Using a standard PostgreSQL database for data storage.

Ponder provides the core indexing engine, while this repository contains the necessary business logic for processing and enriching Doppler protocol data.

## Core Concepts

### Supported Chains

The indexer is configured to support the following networks:

| Name          | Chain ID | Purpose                               |
| ------------- | :------- | ------------------------------------- |
| **Mainnet**   | `1`      | ETH Price Oracle                      |
| **Base**      | `8453`   | V2, V3, V4 Protocol Indexing          |
| **Unichain**  | `130`    | V2, V3, V4 Protocol Indexing          |
| **Ink**       | `57073`  | V2, V3, V4 Protocol Indexing          |
| **Base Sepolia** | `84532`  | V2, V3, V4 Protocol Indexing (Testnet) |

The configuration is modular, making it straightforward to add or remove chains. See the [Development Guide](development.md) for more details.

### Supported Protocols & Events

The indexer listens to a variety of events across different protocol versions to build a complete picture of market activity.

| Contract                    | Event           | Protocol | Description                                          |
| --------------------------- | --------------- | :------- | ---------------------------------------------------- |
| `Airlock`                   | `Migrate`       | V2       | A V3 pool migrating its liquidity to a new V2 pool.  |
| `UniswapV2Pair`             | `Swap`          | V2       | A trade on a Uniswap V2-style pair.                  |
| `UniswapV3Initializer`      | `Create`        | V3       | Creation of a new Doppler V3 pool.                   |
| `UniswapV3Pool`             | `Mint`, `Burn`  | V3       | Adding or removing concentrated liquidity.           |
| `UniswapV3Pool`             | `Swap`          | V3       | A trade on a Doppler V3 pool.                        |
| `UniswapV4Initializer`      | `Create`        | V4       | Creation of a new Doppler V4 pool.                   |
| `UniswapV4Pool`             | `Swap`          | V4       | A trade on a Doppler V4 pool with hooks.             |
| `DERC20`, `V4DERC20`        | `Transfer`      | V3/V4    | DERC20 token transfers, used to track user balances. |
| (Block Handlers)            | `block`         | -        | Periodic tasks for oracle updates and data refresh.  |

## Database Schema Overview

The database schema is defined in `ponder.schema.ts`. It is designed to store normalized data from all supported protocols.

### Core Entities

-   `user`: Represents a user's wallet address.
-   `token`: Detailed information about an ERC20 token, including metadata, total supply, and market data.
-   `asset`: Represents a DERC20 token in the context of its specific Doppler pool, linking it to governance, migrators, etc.
-   `pool`: The central entity for a liquidity pool, storing aggregated data like price, liquidity, volume, and references to its tokens. This table holds data for V3 and V4 pools.
-   `v2Pool`: A specific table for V2 pools that are created via migration, linked back to a `pool` entity.
-   `swap`: A record of an individual trade, including amounts, user, price, and USD value.
-   `position`: For V3 concentrated liquidity, tracking a user's liquidity position within specific tick ranges.

### Time-Series Data

-   `eth_price`: Stores the price of ETH in USD, fetched periodically from a Chainlink oracle.
-   `hour_bucket_usd`: Aggregated OHLC (Open, High, Low, Close) price data in USD for each pool, bucketed by the hour.
-   `daily_volume`: Tracks rolling 24-hour trading volume for each pool.

## Quirks & Limitations

### Quote Currency

**The indexer is fundamentally designed to work with pools quoted in ETH or WETH.**

-   All USD price calculations (`priceUsd`, `liquidityUsd`, `volumeUsd`) are derived by converting the quote currency (ETH) value to USD using the Chainlink ETH/USD oracle price.
-   Pools that use a different numeraire (e.g., USDC) are not fully supported for USD-denominated analytics. While their data will be indexed, USD values may be incorrect or zero.
-   The system identifies the quote token as ETH by checking if its address is the `zeroAddress` or the configured WETH address for that chain.

### Token Metadata Fetching

The indexer attempts to fetch and store metadata (name, symbol, image) for each token from its `tokenURI`.

-   **Supported Formats**:
    1.  **IPFS**: URIs starting with `ipfs://` are resolved using a Pinata gateway. The indexer looks for an `image` or `image_hash` field in the returned JSON.
    2.  **HTTP(S)**: Direct HTTP/S links are fetched.

-   **Retry Mechanism for Failed Fetches**:
    -   If fetching metadata fails, the token is added to a `pending_token_images` table in the database.
    -   A periodic block handler (`PendingTokenImagesBase:block`) runs every 50 blocks.
    -   This handler retries fetching the images for pending tokens.
    -   Each token has a 5-minute cooldown between retries and a maximum of 10 retry attempts before being abandoned.
    -   This ensures that transient network issues or delayed metadata uploads don't prevent images from being indexed eventually.