---
name: grid-query
description: >
  Guide for querying The Grid's Web3 database via MCP tools. Contains
  search patterns, filtering syntax, ordering, and relationship queries for
  profiles, products, assets, tags, and smart contracts. Trigger when the user
  asks to search The Grid, look up Web3 entities, find blockchain projects,
  query profiles/products/assets, or needs structured crypto/blockchain data.
---

# Grid Search — Query Guide

This skill provides the query patterns and field reference for searching
The Grid's Web3 database using the MCP tools.

The Grid contains structured data on 3,000+ Web3 profiles (companies,
protocols, DAOs), 6,400+ products (DEXs, wallets, bridges), 1,600+ assets
(tokens, coins, stablecoins), and the relationships between them — plus
tags, media, attributes, and profile hierarchies.

## Tools

| Tool | Use When |
|------|----------|
| `mcp__grid__find` | Quick search by name, type, or status — no GraphQL needed |
| `mcp__grid__query` | Custom GraphQL — complex filters, joins, aggregations |
| `mcp__grid__get-tgs` | Schema discovery — list lenses, browse enums, get full JSON schemas |

### When to Use Which

- **Start with `find`** for simple lookups ("find Uniswap", "show live wallets")
- **Use `query`** when you need joins, nested fields, specific columns, or multi-entity queries
- **Use `get-tgs`** to discover what's available before writing a query — it lists all lenses, enum values, and field schemas

### `mcp__grid__get-tgs` — Schema Discovery

This tool lets you explore the full data model without writing GraphQL.

| Action | What It Returns | When to Use |
|--------|----------------|-------------|
| `lenses` | All schema lenses with descriptions and table names | "What can I query?" |
| `enums` | All enum types with value counts | "What classification systems exist?" |
| `enum` (+ name) | Full list of values for a specific enum | "What are all the product types?" |
| `schema` (+ name) | Full JSON Schema for a lens or table | "What fields does profiles have?" |

**Examples:**
```
get-tgs(action: "lenses")                    # List all queryable lenses
get-tgs(action: "enums")                     # List all enums with counts
get-tgs(action: "enum", name: "productTypes") # Get all 118 product type values
get-tgs(action: "schema", name: "profiles")   # Get full profile schema
```

**Always use `get-tgs` before guessing at field names or enum values.** It is the source of truth.

## Core Data Model

```
Profile (Company/Protocol/DAO)
  |-- Products (DEX, Wallet, Bridge, L1, L2, AI Agent, etc.)
  |     |-- supportsProducts (links to other products, e.g. which chains)
  |     |-- productAssetRelationships (which assets it interacts with)
  |     |-- productDeployments → smartContractDeployments → smartContracts
  |     +-- URLs, Media
  |-- Assets (Tokens, Coins, Stablecoins)
  |     |-- assetDeployments → smartContractDeployments → smartContracts
  |     +-- derivativeAssets, URLs, Media
  |-- Entities (Legal structures — Foundation, Corporation, etc.)
  |-- Socials (Twitter/X, Discord, Telegram, GitHub, etc.)
  |-- Tags (Event attendance, ecosystem membership, community, tech)
  |-- Media (Icons, Logos, Headers)
  |-- Attributes (Bluechip ratings, ChainIDs)
  +-- Root Relationships (parent/subsidiary, acquisitions)
```

**Key relationships:**
- `root` connects everything to a profile — products, assets, socials all hang off a root
- `supportsProducts` links products to other products (ecosystem membership)
- `productAssetRelationships` links products to assets they interact with
- `rootRelationships` links profiles to each other (parent/child, acquisitions)
- `profileTags` links profiles to tags (events, communities, tech paradigms)

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

## Tag Queries

Tags are a powerful way to find profiles by event attendance, ecosystem membership,
community affiliation, or technology focus. There are 50+ tags across 8 tag types.

**Tag types:** Event, Community, Tech, Paradigm, Geography, Hackathon, Report

**Find profiles by tag name:**
```graphql
profileTags(where: {tag: {name: {_eq: "Breakpoint 25 Attendee"}}}, limit: 20) {
  root {
    profileInfos { name descriptionShort profileType { name } }
  }
  tag { name tagType { name } }
}
```

**Find profiles with a specific tag type (e.g. all event-tagged profiles):**
```graphql
profileTags(where: {tag: {tagType: {slug: {_eq: "event"}}}}, limit: 20) {
  root { profileInfos { name } }
  tag { name }
}
```

**Get all tags for a profile:**
```graphql
profileInfos(where: {name: {_eq: "Jupiter"}}) {
  name
  root {
    profileTags {
      tag { name tagType { name } }
    }
  }
}
```

**Search by partial tag name:**
```graphql
profileTags(where: {tag: {name: {_like: "%Solana%"}}}, limit: 20) {
  root { profileInfos { name } }
  tag { name tagType { name } }
}
```

**Common tag queries:**
- Event attendance: "Breakpoint 2024", "Breakpoint 25 Attendee", "Accelerate '25 Speaker"
- Ecosystem: "Solana", "Ethereum", "Bitcoin", "Starknet", "TRON"
- Community: "SuperteamUK", "SuperteamVN", "Claimed On The Grid"
- Tech/Paradigm: "AI", "DeFi", "Blinks", "Frames (Farcaster)"
- Reports: "Bluechip Report '25"
- Hackathons: "Solana AI Hackathon Winner", "Colosseum Cohort 4"

**To get the full current list of tags, use:**
```
get-tgs(action: "enum", name: "tags")
```

## Profile Relationship Queries

Profiles can be linked to each other via `rootRelationships` — parent/subsidiary,
acquisitions, and brand associations.

**Relationship types:** Acquired by, Associated with, Managed by, Product of

**Find subsidiaries/sub-brands of a profile:**
```graphql
profileInfos(where: {name: {_eq: "Coinbase"}}) {
  name
  root {
    childRootRelationships {
      relationshipType { name }
      childRoot {
        profileInfos { name profileType { name } }
      }
    }
  }
}
```

**Find the parent of a profile:**
```graphql
profileInfos(where: {name: {_eq: "Base"}}) {
  name
  root {
    parentRootRelationships {
      relationshipType { name }
      parentRoot {
        profileInfos { name }
      }
    }
  }
}
```

**Find all acquired profiles:**
```graphql
rootRelationships(
  where: {relationshipType: {slug: {_eq: "acquired_by"}}}
  limit: 20
) {
  parentRoot { profileInfos { name } }
  childRoot { profileInfos { name } }
  relationshipType { name }
}
```

## Blockchain Ecosystem Queries

When a user asks about a blockchain ecosystem (e.g., "What's in the Solana ecosystem?"),
there are **two ways** to find products on that chain.
Always start by finding the chain's Product ID via its Profile.

**Step 1 (always): Profile -> L1/L2 Product ID**

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

Products with smart contracts deployed on the chain. Narrower — only products
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

**`assetSupportType` values:** Native to, Supported by, Governance, Managed by,
Utility, Launched via, Distributed via, Payment Accepted by

## Media Queries

Get logos, icons, and headers for profiles, products, or assets.

```graphql
profileInfos(where: {name: {_eq: "Phantom"}}) {
  name
  root {
    media {
      url
      mediaType { name }
    }
  }
}
```

**Media types:** Icon, Logo on white, Logo on black, Header

## Grid Rank

Profiles have an importance score based on ecosystem connectivity.

```graphql
profileInfos(where: {name: {_eq: "Uniswap"}}) {
  name
  root {
    gridRank { score }
  }
}
```

## Key Rules

- Use `slug` fields for filtering (not IDs) — they are human-readable and stable
- Use `_like` with `%wildcards%` for partial matching, `_eq` for exact
- `_ilike` is NOT supported — use `_like` instead
- Always include `limit` to avoid unbounded queries
- Use `profileInfos` (not `profiles`) and `products` (not `product`)
- Ordering uses `Asc` / `Desc` (no null-handling variants)
- Use `get-tgs` to discover enum values — don't guess at slugs or IDs
- Tags use `profileTags` join table, not a direct field on `profileInfos`

## Reference

For the complete query reference with all patterns, examples, and field
listings, read `references/grid-mcp-guide.md`.
