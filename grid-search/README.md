# Grid Search Plugin

Query The Grid's Web3 database directly from Claude Code. Search 3,000+ profiles
(companies, protocols, DAOs), 6,400+ products (DEXs, wallets, bridges), 1,600+
assets (tokens, coins, stablecoins), and the relationships between them — plus
tags, media, attributes, and profile hierarchies.

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
| `mcp__grid__get-tgs` | Schema discovery — list lenses, browse enums, get full JSON schemas |

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

### `mcp__grid__get-tgs`

Schema discovery tool. Explore the data model, list available lenses,
browse enum values, and get full JSON schemas — all without writing GraphQL.

**Parameters:**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `action` | Yes | `lenses`, `enums`, `enum`, or `schema` |
| `name` | For `enum`/`schema` | Enum name (e.g. "productTypes") or lens name (e.g. "products") |

```
get-tgs action=lenses                      # List all queryable areas
get-tgs action=enums                       # List all enum types with counts
get-tgs action=enum name=productTypes      # Get all 118 product type values
get-tgs action=schema name=profiles        # Get full profile JSON schema
```

## Skills

This plugin includes two auto-triggered skills:

| Skill | Triggers On |
|-------|-------------|
| **grid-query** | Searching The Grid, looking up Web3 entities, querying profiles/products/assets/tags |
| **tgs-enums** | Looking up TGS enum values, taxonomy options, product/asset/profile type IDs, tag values |

### grid-query

Provides query patterns for all entity types including profiles, products, assets,
tags, profile relationships, media, attributes, ecosystem queries, and Grid Rank.
Includes a full reference guide at `skills/grid-query/references/grid-mcp-guide.md`.

### tgs-enums

Lists all 21 queryable enum tables with value counts and ready-to-use query patterns.
Covers product types (118), asset types (16), profile sectors (28), tags (50),
tag types (8), and all status/relationship/deployment enums.

## Usage

Tools are available automatically once installed. Ask naturally:

- "find all live wallets on The Grid"
- "who attended Breakpoint 2025?"
- "what product types exist in TGS?"
- "show me the Solana ecosystem"
- "what are Coinbase's subsidiaries?"
