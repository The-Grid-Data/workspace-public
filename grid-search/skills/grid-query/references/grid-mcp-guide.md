# Grid MCP — Complete Query Reference

Query reference for The Grid's Web3 database via the remote MCP server at `mcp.thegrid.id`.

All queries are executed through the `mcp__grid__query`, `mcp__grid__find`, and
`mcp__grid__get-tgs` tools provided by the grid-search plugin.

---

## Tools

### `mcp__grid__get-tgs` — Schema Discovery

Explore the data model, list available lenses and tables, and get full enum values
without writing GraphQL. **Use this first** when unsure what fields or values exist.

| Action | Parameter | Returns |
|--------|-----------|---------|
| `lenses` | — | All schema lenses with descriptions and their table names |
| `enums` | — | All enum types with value counts |
| `enum` | `name` (e.g. "productTypes") | Full list of const/title/description for an enum |
| `schema` | `name` (e.g. "products") | Full JSON Schema for a lens or table |

**Examples:**

```
# What lenses (query areas) exist?
get-tgs(action: "lenses")

# What enums exist and how many values each has?
get-tgs(action: "enums")

# What are all 118 product types?
get-tgs(action: "enum", name: "productTypes")

# What fields does the products lens have?
get-tgs(action: "schema", name: "products")

# What are the tag types?
get-tgs(action: "enum", name: "tagTypes")

# What tags exist?
get-tgs(action: "enum", name: "tags")
```

### `mcp__grid__query` — Custom GraphQL

Execute custom GraphQL queries against The Grid database.

**When to use:** Complex multi-entity queries, specific field requirements, advanced
filtering, aggregations and counts, relationship traversal.

**Syntax:** Pass GraphQL query strings to the tool:

```graphql
query {
  [entity](filters) {
    field1
    field2
    nestedEntity { field }
  }
}
```

### `mcp__grid__find` — Quick Search

Search for products, assets, or profiles without writing GraphQL. Results are
ordered by Grid Rank (most important first).

| Param | Required | Description |
|-------|----------|-------------|
| `lens` | yes | `"products"`, `"assets"`, or `"profiles"` |
| `name` | no | Filter by name. `%value%` for substring, `value1,value2` for multiple |
| `type` | no | Filter by type (same patterns as name) |
| `status` | no | Filter by status (same patterns as name) |
| `limit` | no | Max results (default: 10) |

**Examples:**

```
find(lens: "products", name: "%Jupiter%")
find(lens: "products", type: "Wallet", status: "Live")
find(lens: "assets", name: "%SOL%")
find(lens: "products", type: "Decentralised Exchange,DEX Aggregator")
find(lens: "profiles", name: "%Uniswap%")
```

**Notes:**
- `find` uses human-readable type/status names (e.g. "Wallet", "Live"), not slugs
- Only supports `products`, `assets`, `profiles` — for tags, socials, relationships,
  use `mcp__grid__query`

---

## Data Model

### Lenses (Query Areas)

| Lens | Description | Key Tables |
|------|-------------|------------|
| `profiles` | Organizations, companies, protocols, DAOs | profileInfos, profileTags, tags, tagTypes |
| `products` | Offerings — DEX, Wallet, Bridge, L1, L2, etc. | products, supportsProducts |
| `assets` | Tokens, coins, stablecoins | assets, derivativeAssets |
| `socials` | Social media accounts | socials |
| `urls` | Web links for all entities | urls, urlTypes |
| `media` | Logos, icons, headers | media, mediaTypes |
| `attributes` | Key-value metadata (ratings, ChainIDs) | attributes, attributeTypes |
| `entities` | Legal structures | entities, entityTypes |
| `productAssetRelationships` | Product-asset links | productAssetRelationships |
| `smartContractDeployments` | On-chain deployments | smartContractDeployments, smartContracts |
| `internalHelpers` | Grid Rank, profile relationships, roots | gridRank, rootRelationships, roots |

### Entity Relationships

```
Profile (profileInfos)
  |-- root (the canonical identity anchor)
  |     |-- products
  |     |-- assets
  |     |-- socials
  |     |-- entities
  |     |-- media
  |     |-- profileTags -> tags
  |     |-- gridRank
  |     |-- childRootRelationships -> childRoot -> profileInfos
  |     +-- parentRootRelationships -> parentRoot -> profileInfos
  |
Product (products)
  |-- productType, productStatus
  |-- supportsProducts (ecosystem membership)
  |-- productDeployments -> smartContractDeployment -> smartContracts
  |-- productAssetRelationships -> asset
  |-- urls, media
  +-- root -> profileInfos (parent profile)

Asset (assets)
  |-- assetType, assetStatus
  |-- assetDeployments -> smartContractDeployment -> smartContracts
  |-- derivativeAssets
  |-- urls, media
  +-- root -> profileInfos (parent profile)
```

### Best Practices for Filtering

**ALWAYS use `slug` fields instead of IDs for enum filtering:**

DO: `{productType: {slug: {_eq: "decentralised_exchange"}}}`

DON'T: `{productTypeId: {_eq: "25"}}`

**Why:** Slugs are human-readable, stable, self-documenting, and don't get truncated.

**Discover slugs with `get-tgs`:**

```
get-tgs(action: "enum", name: "productTypes")   # All product type slugs
get-tgs(action: "enum", name: "assetTypes")      # All asset type slugs
get-tgs(action: "enum", name: "profileSectors")  # All sector slugs
```

Or via GraphQL:

```graphql
productTypes { name slug }
assetTypes { name slug }
productStatuses { name slug }
profileStatuses { name slug }
```

---

## Search Patterns

### Pattern 1: Search by Profile Name

```graphql
# Direct match
profileInfos(where: {name: {_eq: "Jupiter"}}) {
  id name descriptionShort
  profileType { name }
  profileSector { name }
  profileStatus { name }
  urls { url urlType { name } }
}

# Fuzzy match
profileInfos(where: {name: {_like: "%Jup%"}})
```

### Pattern 2: Search by Product Name

```graphql
products(where: {name: {_like: "%Jupiter%"}}, limit: 5) {
  id name
  productType { name }
  root { profileInfos { name } }
}
```

### Pattern 3: Search by Asset Ticker or Name

```graphql
# By ticker
assets(where: {ticker: {_eq: "SOL"}}) {
  id name ticker description
  assetType { name }
  assetStatus { name }
}

# By name
assets(where: {name: {_like: "%Solana%"}})

# With deployments (contract addresses)
assets(where: {ticker: {_eq: "USDC"}}) {
  name ticker
  assetDeployments {
    smartContractDeployment {
      deployedOnProduct { name }
      assetStandard { name }
      smartContracts { address }
    }
  }
}
```

### Pattern 4: Search by Social Media

```graphql
# Find by social handle
socials(where: {name: {_like: "%phantom%"}}, limit: 5) {
  id name
  socialType { name }
  root { profileInfos { name } }
}

# Get all socials for a profile
profileInfos(where: {name: {_eq: "Phantom"}}) {
  name
  root {
    socials {
      socialType { name }
      name
      socialStatus { name }
    }
  }
}
```

### Pattern 5: Search by Tags

Tags categorize profiles by events, ecosystems, communities, tech paradigms, and more.

```graphql
# Find profiles with a specific tag
profileTags(where: {tag: {name: {_eq: "Breakpoint 25 Attendee"}}}, limit: 20) {
  root { profileInfos { name descriptionShort profileType { name } } }
  tag { name tagType { name } }
}

# Find profiles by tag type (e.g. all hackathon participants)
profileTags(where: {tag: {tagType: {slug: {_eq: "hackathon"}}}}, limit: 20) {
  root { profileInfos { name } }
  tag { name }
}

# Search tags by partial name
profileTags(where: {tag: {name: {_like: "%Solana%"}}}, limit: 20) {
  root { profileInfos { name } }
  tag { name tagType { name } }
}

# Get all tags for a specific profile
profileInfos(where: {name: {_eq: "Jupiter"}}) {
  name
  root {
    profileTags { tag { name tagType { name } } }
  }
}

# Count profiles per tag
tags(limit: 50) {
  name
  tagType { name }
  profileTagsAggregate { _count }
}
```

**Tag types:** Event, Community, Tech, Paradigm, Geography, Hackathon, Report

**To get the full tag list:** `get-tgs(action: "enum", name: "tags")`

### Pattern 6: Search by URL

```graphql
# Find profiles by domain
profileInfos(where: {urls: {url: {_like: "%solana.com%"}}}) {
  name
  urls { url urlType { name } }
}

# Find products by domain
products(where: {urls: {url: {_like: "%solana.com%"}}}) {
  name
  urls { url urlType { name } }
  root { profileInfos { name } }
}
```

---

## Profile Relationship Queries

Profiles can be linked hierarchically via `rootRelationships`.

**Relationship types:** Acquired by, Associated with, Managed by, Product of

### Find Children (Subsidiaries, Sub-brands)

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

### Find Parent

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

### Find All Acquisitions

```graphql
rootRelationships(
  where: {relationshipType: {slug: {_eq: "acquired_by"}}}
  limit: 20
) {
  parentRoot { profileInfos { name } }
  childRoot { profileInfos { name } }
}
```

---

## Blockchain Ecosystem Queries

### Step 1: Find the Chain's Product ID

```graphql
profileInfos(where: {name: {_eq: "Solana"}}) {
  name
  root {
    products(where: {productType: {slug: {_in: ["l1", "l2"]}}}) {
      id name
    }
  }
}
# Result: Solana Mainnet -> id: "22"
```

### Method A — `supportsProducts` (broad ecosystem view)

Products that support the chain in any way — even without on-chain deployments.

```graphql
products(
  where: { supportsProducts: { supportsProductId: {_eq: "22"} } }
  limit: 20
) {
  name
  productType { name slug }
  root { profileInfos { name } }
}
```

### Method B — `productDeployments` (on-chain only)

Products with smart contracts deployed on the chain.

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

### Counting Product Types in an Ecosystem

```graphql
productTypes {
  name
  productsAggregate(
    filter_input: {
      where: {supportsProducts: {supportsProductId: {_eq: "22"}}}
    }
  ) { _count }
}
```

---

## Asset Ecosystem Queries

### Products That Interact with an Asset

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

**`assetSupportType` values:**
- **Native to** — blockchain's native token (e.g. SOL for Solana)
- **Supported by** — protocol-level integration (listing, collateral, oracle feed)
- **Governance** — voting/governance token
- **Managed by** — token issuer
- **Utility** — required for product usage / primary payment method
- **Launched via** — asset was issued via this product
- **Distributed via** — distribution managed by a third-party product
- **Payment Accepted by** — can receive as payment, including via gateways

---

## Media Queries

Get logos, icons, and header images for any entity.

**Media types:** Icon, Logo on white, Logo on black, Header

```graphql
# Get all media for a profile
profileInfos(where: {name: {_eq: "Phantom"}}) {
  name
  root {
    media { url mediaType { name } }
  }
}

# Get icons specifically
profileInfos(where: {name: {_eq: "Phantom"}}) {
  name
  root {
    media(where: {mediaType: {slug: {_eq: "icon"}}}) {
      url
    }
  }
}
```

---

## Attribute Queries

Attributes are key-value metadata for entities (e.g. Bluechip stablecoin ratings, ChainIDs).

```graphql
# Get attributes for a profile
attributes(
  where: {attributeType: {slug: {_eq: "bluechip_stablecoin_rating"}}}
  limit: 20
) {
  value
  attributeType { name }
}
```

**Current attribute types:** Bluechip Stablecoin Rating, ChainID - CAIP-2

Use `get-tgs(action: "enum", name: "attributeTypes")` to check for new ones.

---

## Grid Rank

Importance scoring based on ecosystem connectivity (how many other profiles, products,
and assets a profile is linked to). Higher is better.

```graphql
profileInfos(where: {name: {_eq: "Uniswap"}}) {
  name
  root { gridRank { score } }
}
```

---

## Ordering Results

### Supported Order Values

- **`Asc`** — Ascending (A-Z, 0-9, oldest first)
- **`Desc`** — Descending (Z-A, 9-0, newest first)

The API does NOT support `Asc_nulls_first/last` or `Desc_nulls_first/last`.

### Examples

```graphql
# Order by name
products(order_by: { name: Asc }, limit: 10) {
  name productType { name }
}

# Order by date (newest first)
products(
  where: {productStatus: {slug: {_eq: "live"}}}
  order_by: { launchDate: Desc }
  limit: 5
) {
  name launchDate productType { name }
}

# Multiple field ordering
products(
  order_by: [
    { productType: { name: Asc } }
    { name: Asc }
  ]
  limit: 20
) {
  name productType { name }
}

# Nested field ordering
products(
  order_by: { root: { profileInfos: { name: Asc } } }
  limit: 10
) {
  name
  root { profileInfos { name } }
}
```

---

## Common Queries

### Full Profile Details

```graphql
profileInfos(where: {name: {_eq: "Phantom"}}) {
  name logo tagLine descriptionShort descriptionLong
  profileType { name } profileSector { name } profileStatus { name }
  foundingDate
  urls { url urlType { name } }
  root {
    slug
    gridRank { score }
    products {
      name description
      productType { name } productStatus { name }
      launchDate
    }
    assets {
      name ticker
      assetType { name } assetStatus { name }
    }
    socials {
      socialType { name } name socialStatus { name }
    }
    profileTags { tag { name tagType { name } } }
    media { url mediaType { name } }
  }
}
```

### List Products by Type

```graphql
products(where: {productType: {slug: {_eq: "decentralised_exchange"}}}, limit: 5) {
  name description
  productStatus { name slug }
  productType { name slug }
  root { profileInfos { name logo } }
}
```

**Common product type slugs:**
- DEX: `decentralised_exchange`
- Wallet: `wallet`
- Bridge: `bridge`
- Oracle: `oracle`
- RPC Provider: `rpc_provider`
- L1: `l1`, L2: `l2`
- AI Agent: `ai_agent`
- AI Agent Platform: `ai_agent_platform`

There are 118 product types — use `get-tgs(action: "enum", name: "productTypes")` for the full list.

### Profile Discovery by Sector

```graphql
profileInfos(
  where: {profileSector: {slug: {_eq: "finance"}}}
  limit: 20
) {
  name descriptionShort
  profileSector { name slug }
  root {
    productsAggregate { _count }
    assetsAggregate { _count }
  }
}
```

There are 28 profile sectors — use `get-tgs(action: "enum", name: "profileSectors")` for the full list.

### Count Products by Type

```graphql
productTypes(limit: 50) {
  name definition
  productsAggregate { _count }
}
```

### Social Media Inventory

```graphql
profileInfos(where: {name: {_eq: "Uniswap"}}) {
  name
  root {
    socials {
      socialType { name }
      name
      urls { url }
      socialStatus { name }
    }
  }
}
```

### Profile Comparison

```graphql
{
  phantom: profileInfos(where: {name: {_eq: "Phantom"}}) {
    name
    root {
      products { name productType { name slug } productStatus { name slug } }
    }
  }
  metamask: profileInfos(where: {name: {_eq: "MetaMask"}}) {
    name
    root {
      products { name productType { name slug } productStatus { name slug } }
    }
  }
}
```

---

## Advanced Examples

### Multi-Criteria Product Search

```graphql
products(
  where: {
    _and: [
      {name: {_like: "%swap%"}},
      {productStatus: {slug: {_eq: "live"}}},
      {_or: [
        {productType: {slug: {_eq: "decentralised_exchange"}}},
        {productType: {slug: {_eq: "dex_aggregator"}}}
      ]}
    ]
  }
) {
  name description
  productType { name slug }
  productStatus { name slug }
  launchDate
}
```

### Cross-Entity: L2 Blockchains with Native Tokens

```graphql
products(where: {productType: {slug: {_eq: "l2"}}}) {
  name description
  productType { name slug }
  root {
    assets(where: {assetTypeId: {_eq: "1"}}) {
      name ticker description
    }
  }
}
```

### Full-Featured Query with Ordering

```graphql
products(
  where: {
    _and: [
      {name: {_like: "%swap%"}},
      {productStatus: {slug: {_eq: "live"}}},
      {launchDate: {_is_null: false}}
    ]
  }
  order_by: [{ launchDate: Desc }, { name: Asc }]
  limit: 5
) {
  name launchDate description
  productType { name slug }
  productStatus { name slug }
  root { profileInfos { name } }
}
```

### Event/Conference Analysis via Tags

```graphql
# Who attended Breakpoint 2025 and what do they do?
profileTags(where: {tag: {name: {_eq: "Breakpoint 25 Attendee"}}}, limit: 50) {
  root {
    profileInfos {
      name descriptionShort
      profileSector { name }
      profileType { name }
    }
  }
}
```

### Stablecoin Ecosystem Query

```graphql
# Find all products that accept payments in USDC
productAssetRelationships(
  where: {
    _and: [
      {asset: {ticker: {_eq: "USDC"}}},
      {assetSupportType: {slug: {_eq: "payment_accepted_by"}}}
    ]
  }
  limit: 20
) {
  product {
    name productType { name }
    root { profileInfos { name } }
  }
}
```

---

## Filter Reference

| Filter | Syntax | Example |
|--------|--------|---------|
| Exact match | `{name: {_eq: "Jupiter"}}` | Name equals |
| Contains | `{name: {_like: "%wallet%"}}` | Substring search |
| In list | `{slug: {_in: ["l1", "l2"]}}` | Multiple values |
| AND | `{_and: [{...}, {...}]}` | All conditions |
| OR | `{_or: [{...}, {...}]}` | Any condition |
| Null check | `{field: {_is_null: false}}` | Non-null only |
| By slug | `{productType: {slug: {_eq: "wallet"}}}` | Recommended for enums |

---

## Troubleshooting

### Common Errors

**"Field not found":**
- Use `profileInfos` not `profiles`
- Use `products` not `product`
- Use nested paths: `root { profileInfos { name } }`
- Check field names with `get-tgs(action: "schema", name: "...")`

**"Invalid where clause":**
- Ensure proper nesting: `{field: {_operator: value}}`
- Use `_like` with wildcards: `"%value%"`
- `_ilike` is NOT supported — use `_like`

**No results:**
1. Broaden search with `%partial%` instead of exact match
2. Check spelling carefully
3. Verify you're querying the right entity type
4. Use `get-tgs` to confirm valid enum values

### Known Limitations

1. `_ilike` (case-insensitive) is not supported — use `_like`
2. `find` only supports products, assets, profiles — use `query` for tags, socials, etc.
3. Direct URL content filtering is limited — search by name first, then check URLs
4. Social search works by handle name, not full URL

---

## Quick Reference

```graphql
# Find by name
profileInfos(where: {name: {_eq: "Jupiter"}})
products(where: {name: {_like: "%Jupiter%"}})
assets(where: {ticker: {_eq: "SOL"}})

# Filter by slug (RECOMMENDED)
products(where: {productType: {slug: {_eq: "decentralised_exchange"}}})
assets(where: {assetType: {slug: {_eq: "stablecoin"}}})
products(where: {productStatus: {slug: {_eq: "live"}}})

# Tags
profileTags(where: {tag: {name: {_eq: "Breakpoint 25 Attendee"}}})
profileTags(where: {tag: {tagType: {slug: {_eq: "event"}}}})

# Profile relationships
root { childRootRelationships { childRoot { profileInfos { name } } } }
root { parentRootRelationships { parentRoot { profileInfos { name } } } }

# Media
root { media { url mediaType { name } } }

# Grid Rank
root { gridRank { score } }

# Social search
socials(where: {name: {_like: "%handle%"}})

# Multi-criteria
_and: [{...}, {...}]
_or: [{...}, {...}]

# Pattern matching
_like: "%value%" (contains)
_eq: "exact" (exact match)
_in: ["a", "b"] (multiple values)
```

```
# Schema discovery (use get-tgs)
get-tgs(action: "lenses")                       # All queryable areas
get-tgs(action: "enums")                         # All enum types
get-tgs(action: "enum", name: "productTypes")    # Specific enum values
get-tgs(action: "schema", name: "products")      # Full field schema
```
