---
hidden: true
icon: laptop
---

# Development

## Configuration

The project features a highly modular and type-safe configuration system located in `src/config/`. This structure replaced a monolithic approach, making the system easier to maintain and scale.

* **Chains**: `src/config/chains/` - Each chain has its own configuration file defining its ID, RPC, start blocks, and contract addresses.
* **Contracts**: `src/config/contracts/` - Contract definitions are generated dynamically based on the chain configurations. This avoids manual repetition.
* **Blocks**: `src/config/blocks/` - Defines periodic tasks (like oracle updates) that run at set block intervals.

This modularity means that adding a new chain or contract is a simple, isolated task with minimal risk to existing configurations.

## Adding a New Chain

1. Create a new file `src/config/chains/newchain.ts` with the `ChainConfig` structure.
2. Add all relevant contract addresses for the new chain.
3. Export the config and add it to the `chainConfigs` object in `src/config/chains/index.ts`.
4. Add the corresponding RPC environment variable to `.env.example` and your local `.env.local`.

## Adding a New Contract

1. Add the contract ABI file to the appropriate directory in `src/abis/`.
2. Update the contract configuration generators in `src/config/contracts/` to include the new contract.
3. If the contract is a factory, create a new factory helper in `src/config/contracts/factories.ts`.
4. Add the new event handler logic in the relevant `src/indexer/indexer-v*.ts` file.

## Code Generation

After making changes to the database schema (`ponder.schema.ts`), you must regenerate the Ponder types.

```bash
pnpm codegen
```

This ensures that all database interactions and API types are updated to reflect your schema changes.
