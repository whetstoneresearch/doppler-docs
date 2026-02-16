---
icon: brackets-curly
---

# API Usage

The indexer exposes the indexed data through a GraphQL API and a RESTful search endpoint.

{% hint style="success" %}
[Whetstone Research](https://whetstone.cc) hosts a free endpoint that supports Base Sepolia for development.&#x20;

[https://testnet-indexer.doppler.lol/](https://testnet-indexer.doppler.lol/)
{% endhint %}

## GraphQL API

The primary way to query data is through the GraphQL endpoint, available at `/graphql`. It is strongly typed and supports complex queries, filtering, and pagination.

### Example Queries

1.  **Fetch top 5 pools on Base by USD liquidity:**

    ```graphql
    query TopPoolsByLiquidity {
      pools(
        where: { chainId: 8453 }
        orderBy: "dollarLiquidity"
        orderDirection: "desc"
        limit: 5
      ) {
        address
        dollarLiquidity
        volumeUsd
        baseToken {
          symbol
        }
        quoteToken {
          symbol
        }
      }
    }
    ```
2.  **Get recent swaps for a specific pool:**

    ```graphql
    query RecentSwaps {
      swaps(
        where: { pool: "0x..." }
        orderBy: "timestamp"
        orderDirection: "desc"
        limit: 10
      ) {
        txHash
        timestamp
        type
        amountIn
        amountOut
        usdPrice
      }
    }
    ```
3.  **Fetch detailed information for a single token:**

    ```graphql
    query TokenDetails {
      token(id: "0x...") {
        address
        name
        symbol
        decimals
        image
        volumeUsd
        holderCount
        pool {
          address
          price
        }
      }
    }
    ```

## REST API

A simple REST endpoint is available for searching tokens.

### Search Endpoint

* **URL**: `/search/:query`
* **Method**: `GET`
* **Description**: Searches for tokens by name, symbol, or address.
* **URL Parameters**:
  * `query`: The search term (e.g., "MyToken", "MTK", or "0x...").
* **Query Parameters**:
  * `chain_ids`: A comma-separated list of chain IDs to filter the search (e.g., `?chain_ids=8453,130`).

**Example Usage:**

```bash
# Search for "doppler" token on Base (chain ID 8453)
curl "http://localhost:42069/search/doppler?chain_ids=8453"

# Search by contract address on Base and Ink
curl "http://localhost:42069/search/0x123...abc?chain_ids=8453,57073"
```

## Direct SQL Access (Development)

For development and debugging, you can connect directly to the PostgreSQL database. Use `pnpm db shell` to get an interactive psql session. You can also use any standard SQL client with the connection string from your `.env.local` file.
