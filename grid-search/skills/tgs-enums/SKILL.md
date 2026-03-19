---
name: tgs-enums
description: >
  Look up all TGS enum values (product types, asset types, statuses, social types,
  URL types, etc.) from The Grid database. Trigger when the user asks about TGS enums,
  taxonomy values, valid IDs, product type definitions, or needs to know what classification
  options exist in The Grid. Also trigger when classifying a product, asset, or profile
  and needing the correct enum ID.
---

# TGS Enum Lookup

Query The Grid database to get the current, authoritative list of all TGS enum values.
Use `mcp__grid__query` to fetch any of the tables below.

## Available Enum Tables

| Table | Fields | Use For |
|-------|--------|---------|
| `productTypes` | id, name, slug, definition | Classifying products (DEX, Wallet, L1, etc.) |
| `productStatuses` | id, name, slug | Product lifecycle (Live, In Development, etc.) |
| `assetTypes` | id, name, slug | Classifying assets (Currency, Stablecoin, Governance, etc.) |
| `assetStatuses` | id, name, slug | Asset lifecycle (Active, Inactive, etc.) |
| `assetStandards` | id, name, slug | Token standards (ERC-20, SPL, BEP-20, etc.) |
| `profileTypes` | id, name, slug | Classifying profiles (Company, DAO, Project, etc.) |
| `profileStatuses` | id, name, slug | Profile lifecycle (Active, Announced, Closed, etc.) |
| `profileSectors` | id, name, slug | Industry sectors (Finance, Gaming, Infrastructure, etc.) |
| `socialTypes` | id, name, slug | Social platforms (Twitter/X, Discord, Telegram, etc.) |
| `urlTypes` | id, name, slug | URL categories (Main, Blog, Documentation, etc.) |
| `assetSupportTypes` | id, name, slug | Product-asset relationships (Native to, Supported by, etc.) |
| `deploymentTypes` | id, name, slug | Smart contract deployment types (Mint, Pool, General) |
| `entityTypes` | id, name, slug | Legal entity types (Foundation, Corporation, etc.) |

## How to Query

**Get all values from a single table:**

```graphql
query { productTypes(limit: 100) { id name slug definition } }
```

**Get multiple tables in one request (recommended):**

```graphql
query {
  productTypes(limit: 200) { id name slug definition }
  assetTypes(limit: 100) { id name slug }
  profileTypes(limit: 50) { id name slug }
  profileSectors(limit: 100) { id name slug }
}
```

**Get all status enums at once:**

```graphql
query {
  productStatuses(limit: 50) { id name slug }
  assetStatuses(limit: 50) { id name slug }
  profileStatuses(limit: 50) { id name slug }
}
```

**Get all relationship and deployment enums:**

```graphql
query {
  assetSupportTypes(limit: 50) { id name slug }
  deploymentTypes(limit: 50) { id name slug }
  assetStandards(limit: 100) { id name slug }
  entityTypes(limit: 50) { id name slug }
}
```

**Get social and URL type enums:**

```graphql
query {
  socialTypes(limit: 50) { id name slug }
  urlTypes(limit: 50) { id name slug }
}
```

## Tips

- Use `slug` values for filtering in queries (e.g., `productType: {slug: {_eq: "wallet"}}`)
- Use `id` values when writing TGS JSON output
- `productTypes` includes a `definition` field — useful for classification decisions
- Combine multiple enum queries in a single request to save round-trips
- These tables are the source of truth — always query them rather than relying on memorized values
