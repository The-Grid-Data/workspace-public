# Grid Search Plugin

Query The Grid's Web3 database directly from Claude Code. Search 3,000+ profiles
(companies, protocols, DAOs), 6,400+ products (DEXs, wallets, bridges), 1,600+
assets (tokens, coins, stablecoins), and the relationships between them.

Connects to The Grid's hosted MCP server at `mcp.thegrid.id`.

## Install

```
/plugin install grid-search@workspace
```

## Requirements

None — connects to a remote MCP server. No local dependencies needed.

## Tools

| Tool | Description |
|------|-------------|
| `mcp__grid__query` | Execute custom GraphQL queries — complex filters, multi-entity joins, aggregations |
| `mcp__grid__find` | Quick search for products, assets, or profiles by name, type, or status — no GraphQL needed |

### `mcp__grid__query`

Full GraphQL access to The Grid API. Supports filtering (`_eq`, `_like`, `_in`),
ordering (`Asc`/`Desc`), pagination (`limit`/`offset`), aggregates (`_count`),
and relationship traversal across all tables.

**Parameters:** `query` (GraphQL string, required), `variables` (object, optional)

```graphql
query {
  products(where: {productType: {slug: {_eq: "wallet"}}}, limit: 5) {
    name productType { name } root { profileInfos { name } }
  }
}
```

### `mcp__grid__find`

Simplified search that returns products, assets, or profiles without writing GraphQL.
Results are ordered by Grid Rank (most important first).

**Parameters:**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `lens` | Yes | `products`, `assets`, or `profiles` |
| `name` | No | Name search (partial match) |
| `type` | No | Filter by type name (e.g. "Wallet", "DEX") |
| `status` | No | Filter by status name (e.g. "Live", "Active") |
| `limit` | No | Max results (default 10) |

```
find lens=products name=Uniswap type=DEX limit=5
```

Note: `find` uses human-readable names for type/status, not slugs.
Supports `products`, `assets`, and `profiles` lenses.

## Skills

This plugin includes two auto-triggered skills:

| Skill | Triggers On |
|-------|-------------|
| **grid-query** | Searching The Grid, looking up Web3 entities, querying profiles/products/assets |
| **tgs-enums** | Looking up TGS enum values, taxonomy options, product/asset/profile type IDs |

### grid-query

Provides query patterns, filtering syntax, ordering, and relationship queries.
Includes a full reference guide at `skills/grid-query/references/grid-mcp-guide.md`
with blockchain network lookup steps, known naming discrepancies, and data analysis strategies.

### tgs-enums

Lists all 13 queryable enum tables (productTypes, assetTypes, profileTypes,
profileSectors, profileStatuses, productStatuses, assetStatuses, assetStandards,
socialTypes, urlTypes, assetSupportTypes, deploymentTypes, entityTypes) with
ready-to-use query patterns grouped by use case.

## Usage

Tools are available automatically once installed. Use them directly or via the
Vertex plugin's `/vertex:research` command.

Ask naturally — "find all live wallets on The Grid" or "what product types exist
in TGS" — and the skills will trigger automatically with the right query patterns.
