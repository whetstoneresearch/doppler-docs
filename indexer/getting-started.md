---
hidden: true
icon: rocket
---

# Getting Started

This section guides you through setting up and running the indexer on your local machine.

## Prerequisites

* **Node.js**: Version `18.14` or higher.
* **pnpm**: The project uses `pnpm` for package management. Install it with `npm install -g pnpm`.
* **Docker** & **Docker Compose**: Required to run the PostgreSQL database.

## Installation

Clone the repository and install the dependencies:

```bash
git clone <repository_url>
cd doppler-v3-indexer
pnpm install
```

## Environment Setup

The indexer requires RPC endpoints for the chains it indexes.

1.  **Create an environment file**:

    ```bash
    cp .env.example .env.local
    ```
2.  **Populate the environment file**:\
    You'll need to add your RPC URLs for each supported network. The `ponder.config.ts` file maps these environment variables to the correct chains.

    ```.env.local
    # Mainnet (for ETH price oracle)
    PONDER_RPC_URL_1="https://eth.drpc.org"

    # Base
    PONDER_RPC_URL_8453="https://base.drpc.org"

    # Unichain
    PONDER_RPC_URL_130="<your_unichain_rpc_url>"

    # Ink
    PONDER_RPC_URL_57073="<your_ink_rpc_url>"

    # Base Sepolia (Testnet)
    PONDER_RPC_URL_84532="https://sepolia.base.org"

    # Pinata Gateway (for fetching token metadata)
    PINATA_GATEWAY_URL="<your_pinata_gateway_url>"
    PINATA_GATEWAY_KEY="<your_pinata_gateway_key>"
    ```

## Running the Indexer

1.  **Start the Database**:\
    The project includes a `docker-compose.yml` file to spin up a PostgreSQL database with performance optimizations.

    ```bash
    docker-compose up -d doppler-indexer-database
    ```
2.  **Start the Indexer**:\
    Run the development server, which will start the indexing process with hot-reloading.

    ```bash
    pnpm dev
    ```

Upon starting, Ponder will:

* Connect to the database and apply the schema.
* Start fetching and processing events from the `startBlock` defined for each contract.
* Launch the GraphQL and REST API servers.

## Database Management

Ponder provides a set of database management utilities accessible via `pnpm db`.

*   **Open a database shell**:

    ```bash
    pnpm db shell
    ```
*   **Reset the database** (clears all data and re-applies the schema):

    ```bash
    pnpm db reset
    ```
