---
name: tgs-enums
description: >
  Look up all TGS enum values (product types, asset types, statuses, social types,
  URL types, tags, etc.) from The Grid database. Trigger when the user asks about
  TGS enums, taxonomy values, valid IDs, product type definitions, tag values,
  or needs to know what classification options exist in The Grid.
---

# TGS Enum Lookup

Use `mcp__grid__get-tgs` to fetch the current, authoritative list of all TGS enum values.
This is the fastest and most reliable way to discover what classification options exist.

## Quick Start

```
# List all enum types with value counts
get-tgs(action: "enums")

# Get full values for a specific enum
get-tgs(action: "enum", name: "productTypes")
```

## Available Enums

| Enum | Count | Use For |
|------|-------|---------|
| `productTypes` | 118 | Classifying products (DEX, Wallet, L1, AI Agent, etc.) |
| `productStatuses` | 7 | Product lifecycle (Live, In Development, etc.) |
| `assetTypes` | 16 | Classifying assets (Currency, Stablecoin, Governance, etc.) |
| `assetStatuses` | 4 | Asset lifecycle (Active, Inactive, etc.) |
| `assetStandards` | 18 | Token standards (ERC-20, SPL, BEP-20, etc.) |
| `assetSupportTypes` | 9 | Product-asset relationships (Native to, Supported by, Launched via, etc.) |
| `profileTypes` | 10 | Classifying profiles (Company, DAO, Protocol, Network, etc.) |
| `profileStatuses` | 6 | Profile lifecycle (Active, Announced, Closed, etc.) |
| `profileSectors` | 28 | Industry sectors (Finance, Gaming, Infrastructure, etc.) |
| `tags` | 50 | Profile tags (events, ecosystems, communities, tech) |
| `tagTypes` | 8 | Tag categories (Event, Community, Tech, Hackathon, etc.) |
| `socialTypes` | 21 | Social platforms (Twitter/X, Discord, Telegram, etc.) |
| `socialStatuses` | 5 | Social account status (Active, Inactive, Suspended, etc.) |
| `urlTypes` | 24 | URL categories (Main, Blog, Documentation, etc.) |
| `mediaTypes` | 5 | Image types (Icon, Logo on white, Logo on black, Header) |
| `deploymentTypes` | 4 | Smart contract deployment types (Mint, Pool, General) |
| `entityTypes` | 10 | Legal entity types (Foundation, Corporation, etc.) |
| `rootRelationshipTypes` | 5 | Profile relationships (Acquired by, Managed by, Product of, etc.) |
| `countries` | 250 | Country codes (ISO 3166-1 alpha-2) |
| `attributeTypes` | 2 | Metadata attributes (Bluechip Rating, ChainID) |
| `coreTableNames` | 6 | Primary data tables in the schema |

## How to Query

**Get values for a single enum (recommended):**

```
get-tgs(action: "enum", name: "productTypes")
```

Returns const (ID), title (display name), and description for each value.

**Get all enum types and counts:**

```
get-tgs(action: "enums")
```

**Alternative: GraphQL queries** (useful when you need slugs or want to combine with data):

```graphql
# Single table
query { productTypes(limit: 200) { id name slug definition } }

# Multiple tables in one request
query {
  productTypes(limit: 200) { id name slug definition }
  assetTypes(limit: 100) { id name slug }
  profileTypes(limit: 50) { id name slug }
  profileSectors(limit: 100) { id name slug }
}

# All status enums
query {
  productStatuses(limit: 50) { id name slug }
  assetStatuses(limit: 50) { id name slug }
  profileStatuses(limit: 50) { id name slug }
}

# Tags and tag types
query {
  tags(limit: 100) { id name slug tagType { name slug } }
  tagTypes(limit: 20) { id name slug definition }
}
```

## Tips

- Use `slug` values for filtering in queries (e.g., `productType: {slug: {_eq: "wallet"}}`)
- Use `id` / `const` values when writing TGS JSON output
- `productTypes` includes a `definition` field — useful for classification decisions
- `get-tgs` returns enum values directly — no GraphQL needed
- These are the source of truth — always query rather than relying on memorized values
- Enum counts grow over time — always check `get-tgs(action: "enums")` for current counts
