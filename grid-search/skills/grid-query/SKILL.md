---
name: grid-query
description: >
  Guide for querying The Grid's Web3 database via GraphQL MCP tools. Contains
  search patterns, filtering syntax, ordering, and relationship queries for
  profiles, products, assets, and smart contracts. Trigger when the user asks
  to search The Grid, look up Web3 entities, find blockchain projects, query
  profiles/products/assets, or needs structured crypto/blockchain data.
---

# Grid Search — Query Guide

This skill provides the query patterns and field reference for searching
The Grid's Web3 database using the `mcp__the-grid__query` tool.

The Grid contains structured data on 3,000+ Web3 profiles (companies,
protocols, DAOs), 6,400+ products (DEXs, wallets, bridges), 1,600+ assets
(tokens, coins, stablecoins), and the relationships between them.

## Tools

| Tool | Use When |
|------|----------|
| `mcp__grid__query` | Execute custom GraphQL queries — complex filters, multi-entity joins, aggregations |
| `mcp__grid__find` | Quick search for products, assets, or profiles by name, type, or status — no GraphQL needed |

## Core Data Model

```
Profile (Company/Protocol)
  - Products (DEX, Wallet, Bridge, etc.)
      - supportsProducts (links to other products, e.g., which chains it supports)
      - Product-Asset Relationships
      - Product Deployments (Smart Contracts)
      - Product URLs
  - Assets (Tokens, Coins)
      - Asset Deployments
      - Asset URLs
  - Entities (Legal structures)
  - Profile URLs
  - Social Media Accounts
```

## Quick Start Patterns

**Find a profile by name:**
```graphql
profileInfos(where: {name: {_eq: "Phantom"}}) {
  name descriptionShort profileType { name } profileStatus { name }
  urls { url urlType { name } }
}
```

**Find products by type (use slugs, not IDs):**
```graphql
products(where: {productType: {slug: {_eq: "decentralised_exchange"}}}, limit: 10) {
  name productType { name slug } productStatus { name slug }
  root { profileInfos { name } }
}
```

**Find an asset by ticker:**
```graphql
assets(where: {ticker: {_eq: "SOL"}}) {
  name ticker assetType { name } assetStatus { name }
}
```

**Search by social handle:**
```graphql
socials(where: {name: {_like: "%phantom%"}}, limit: 5) {
  name socialType { name }
  root { profileInfos { name } }
}
```

## Blockchain Ecosystem Queries

When a user asks about a blockchain ecosystem (e.g., "What's in the Solana ecosystem?",
"Show me Arbitrum products"), there are **two ways** to find products on that chain.
Always start by finding the chain's Product ID via its Profile.

**Step 1 (always): Profile → L1/L2 Product ID**

```graphql
profileInfos(where: {name: {_eq: "Solana"}}) {
  name
  root {
    products(where: {productType: {slug: {_in: ["l1", "l2"]}}}) {
      id
      name
    }
  }
}
# Result: Solana Mainnet (id: "22")
```

**Method A — `supportsProducts` (broader, recommended first)**

Products declare which other products they support. This captures ecosystem membership
even without on-chain deployments (e.g., data platforms, trading bots, analytics tools).

```graphql
products(
  where: {
    supportsProducts: {
      supportsProductId: {_eq: "22"}
    }
  }
  limit: 20
) {
  name
  productType { name slug }
  root { profileInfos { name } }
}
```

**Method B — `productDeployments` (on-chain only)**

Products with smart contracts deployed on the chain. This is narrower — only products
with verified on-chain deployments, but includes contract addresses.

```graphql
products(
  where: {
    productDeployments: {
      smartContractDeployment: {
        deployedOnProduct: {name: {_eq: "Solana Mainnet"}}
      }
    }
  }
  limit: 20
) {
  name
  productType { name slug }
  productDeployments {
    smartContractDeployment {
      deployedOnProduct { name }
      smartContracts { address }
    }
  }
}
```

**When to use which:**
- **`supportsProducts`** = broad ecosystem view (supports the chain in any way)
- **`productDeployments`** = on-chain footprint (has smart contracts deployed on the chain)
- Use both together for a complete picture
- **Do NOT** just search by profile name or tags — that misses most ecosystem participants

## Asset Ecosystem Queries

To find which products interact with a specific asset, use `productAssetRelationships`.

```graphql
productAssetRelationships(
  where: {asset: {ticker: {_eq: "SOL"}}}
  limit: 20
) {
  product {
    name
    productType { name slug }
    root { profileInfos { name } }
  }
  asset { name ticker }
  assetSupportType { name }
}
```

**`assetSupportType` values:** Native to, Supported by, Governance, Managed by, Utility

## Key Rules

- Use `slug` fields for filtering (not IDs) — they are human-readable and stable
- Use `_like` with `%wildcards%` for partial matching, `_eq` for exact
- `_ilike` is NOT supported — use `_like` instead
- Always include `limit` to avoid unbounded queries
- Use `profileInfos` (not `profiles`) and `products` (not `product`)
- Ordering uses `Asc` / `Desc` (no null-handling variants)

## Reference

For the complete query reference with all patterns, examples, and field
listings, read `references/grid-mcp-guide.md`.
