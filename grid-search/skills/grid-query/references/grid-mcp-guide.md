# Grid MCP — Complete Query Reference

Query reference for The Grid's Web3 database via the remote MCP server at `mcp.thegrid.id`.

All queries are executed through the `mcp__the-grid__query` and `mcp__the-grid__find` tools,
which are provided by the grid-search plugin. No direct API access or local setup needed.

All core query patterns tested and confirmed working. Known limitations documented below.

---

## Core Concepts

### Data Structure

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

### Key Entities

**Profile** = Top-level organization (e.g., "Uniswap", "Aave", "Phantom")

**Product** = Specific offering (e.g., "Uniswap V3", "Aave V3 Lending", "Phantom Wallet")

**Asset** = Digital token/coin (e.g., "UNI", "AAVE", "Bitcoin")

### Relationships

- **Products <-> Assets**: Products support/integrate specific assets (via `productAssetRelationships`)
- **Products <-> Products**: Products support other products, e.g., a DEX supports an L1 chain (via `supportsProducts`)
- **Products <-> Blockchains**: Products deploy on specific networks (via `productDeployments`)
- **Assets <-> Blockchains**: Assets exist on specific chains (via `assetDeployments`)
- **Profiles <-> Entities**: Legal structures behind profiles

**`supportsProducts`** is the key relationship for ecosystem queries. Each product has a
`supportsProducts` array where `supportsProductId` points to the product it supports.
To find all products in a blockchain ecosystem, query products where
`supportsProducts.supportsProductId` equals the chain's product ID.

### Best Practices for Filtering

**ALWAYS use `slug` fields instead of IDs for enum filtering:**

DO: `{productType: {slug: {_eq: "decentralised_exchange"}}}`

DON'T: `{productTypeId: {_eq: "25"}}`

**Why slugs are better:**

- **Human-readable**: "wallet" vs "692"
- **Stable**: Slugs don't change, IDs might
- **No truncation**: Long IDs get cut off, slugs don't
- **Self-documenting**: Code is easier to understand

**How to discover slugs:**

```graphql
# Get all product type slugs
productTypes { name slug }

# Get all asset type slugs
assetTypes { name slug }

# Get all status slugs
productStatuses { name slug }
profileStatuses { name slug }
assetStatuses { name slug }
```

---

## MCP Tools

### `mcp__the-grid__query`

Execute custom GraphQL queries against The Grid database.

**When to use:**

- Complex multi-entity queries
- Specific field requirements
- Advanced filtering with GraphQL operators
- Aggregations and counts

**Syntax:**

Pass GraphQL query strings to the tool. The query body goes inside the tool call:

```graphql
query {
  [entity](filters) {
    field1
    field2
    nestedEntity { field }
  }
}
```

### `mcp__the-grid__find`

Quick search for products, assets, or profiles without writing GraphQL. Supports filtering by
name, type, and status.

**Parameters:**

| Param | Required | Description |
|-------|----------|-------------|
| `lens` | yes | `"products"`, `"assets"`, or `"profiles"` |
| `name` | no | Filter by name. `%value%` for substring, `value1,value2` for multiple, or exact match |
| `type` | no | Filter by type (same patterns as name) |
| `status` | no | Filter by status (same patterns as name) |
| `limit` | no | Max results (default: 10) |

**Examples:**

```
# Find products with "Jupiter" in the name
find(lens: "products", name: "%Jupiter%")

# Find all live wallets
find(lens: "products", type: "Wallet", status: "Live")

# Find assets by ticker-like name
find(lens: "assets", name: "%SOL%")

# Find multiple product types
find(lens: "products", type: "Decentralised Exchange,DEX Aggregator")

# Find profiles by name
find(lens: "profiles", name: "%Uniswap%")
```

**Notes:**
- `lens` supports `"products"`, `"assets"`, and `"profiles"` — for socials or other
  entities, use `mcp__grid__query` with GraphQL instead
- The `type` and `status` filters match against the human-readable name (e.g., "Wallet",
  "Live"), not the slug or ID
- Use `mcp__grid__query` when you need joins, nested fields, relationships, or entities
  beyond products/assets

---

## Search Patterns

### Pattern 1: Search by Profile Name

**VERIFIED WORKING**

**Direct match:**

```graphql
profileInfos(where: {name: {_eq: "Jupiter"}}) {
  id
  name
  descriptionShort
  profileType { name }
  profileSector { name }
  profileStatus { name }
  urls { url urlType { name } }
}
```

**Fuzzy match:**

```graphql
profileInfos(where: {name: {_like: "%Jup%"}})
```

**What you get:**

- Profile ID, name, slug
- Profile type (Company, DAO, Project)
- Sector (Finance, Infrastructure, etc.)
- Status (Active, Inactive, etc.)
- URLs, founding date, descriptions

### Pattern 2: Search by Product Name

**VERIFIED WORKING**

```graphql
products(where: {name: {_like: "%Jupiter%"}}, limit: 5) {
  id
  name
  productType { name }
  root { profileInfos { name } }
}
```

**Returns:**

- Product ID, name, description
- Product type (Wallet, DEX, Bridge, etc.)
- Status (Live, Beta, Discontinued)
- Parent profile information
- Smart contract deployments (if queried)

### Pattern 3: Search by Asset Ticker or Name

**VERIFIED WORKING**

**By ticker:**

```graphql
assets(where: {ticker: {_eq: "SOL"}}) {
  id
  name
  ticker
  description
  assetType { name }
  assetStatus { name }
}
```

**By full name:**

```graphql
assets(where: {name: {_like: "%Solana%"}})
```

**With deployments:**

```graphql
assets(where: {ticker: {_eq: "USDC"}}) {
  name
  ticker
  assetDeployments {
    smartContractDeployment {
      deployedOnProduct { name }
      assetStandard { name }
      smartContracts { address }
    }
  }
}
```

**Returns:**

- Asset ID, name, ticker, icon
- Asset type (Currency, Stablecoin, Governance, etc.)
- Deployment details (blockchain, contract address)
- Parent profile

### Pattern 4: Search by URL

**LIMITED** — Direct URL filtering not fully supported in current schema.

**Workaround — Get profile first, then check URLs:**

```graphql
# Step 1: Find profile by name
profileInfos(where: {name: {_like: "%Jupiter%"}}) {
  id
  name
  urls {
    url
    urlType { name }
  }
}

# Step 2: Manually filter results by URL pattern
```

**Product URL search — Same approach:**

```graphql
products(where: {name: {_like: "%keyword%"}}) {
  name
  urls { url urlType { name } }
}
```

### Pattern 5: Search by Social Media

**VERIFIED WORKING**

**Find by social handle:**

```graphql
socials(where: {name: {_like: "%phantom%"}}, limit: 5) {
  id
  name
  socialType { name }
  root {
    profileInfos { name }
  }
}
```

**Get all socials for a profile:**

```graphql
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

**What you get:**

- Twitter/X, Discord, Telegram, GitHub, etc.
- Social account names and handles
- Social status (Active, Inactive, Suspended)
- Parent profile information

---

## Ordering Results

**VERIFIED WORKING**

The Grid GraphQL API supports ordering query results using the `order_by` parameter.

### Supported Order Values

- **`Asc`** — Ascending order (A to Z, 0 to 9, oldest to newest)
- **`Desc`** — Descending order (Z to A, 9 to 0, newest to oldest)

The API does NOT support `Asc_nulls_first`, `Asc_nulls_last`, `Desc_nulls_first`, or `Desc_nulls_last`.

### Single Field Ordering

```graphql
# Order products by name (ascending)
products(order_by: { name: Asc }, limit: 10) {
  name
  productType { name }
}

# Order assets by ticker (descending)
assets(order_by: { ticker: Desc }, limit: 10) {
  name
  ticker
}

# Order by date (newest first)
products(
  where: {productStatus: {slug: {_eq: "live"}}}
  order_by: { launchDate: Desc }
  limit: 5
) {
  name
  launchDate
  productType { name }
}
```

### Multiple Field Ordering

Order by multiple fields by passing an array. First field has priority.

```graphql
products(
  order_by: [
    { productType: { name: Asc } }
    { name: Asc }
  ]
  limit: 20
) {
  name
  productType { name }
}
```

### Ordering Nested Fields

```graphql
products(
  order_by: { root: { profileInfos: { name: Asc } } }
  limit: 10
) {
  name
  root {
    profileInfos { name }
  }
}
```

### Combining Filters and Ordering

```graphql
# Live DEX products, newest first
products(
  where: {
    _and: [
      {productType: {slug: {_eq: "decentralised_exchange"}}},
      {productStatus: {slug: {_eq: "live"}}}
    ]
  }
  order_by: { launchDate: Desc }
  limit: 10
) {
  name
  launchDate
  productType { name slug }
  productStatus { name slug }
}
```

### Ordering Notes

1. Ordering is case-sensitive by default
2. Null values appear at the end for `Asc` and at the beginning for `Desc`
3. Combine `order_by` with `limit` and `offset` for paginated results

---

## Common Queries

### 1. Get Full Profile Details

**VERIFIED WORKING**

```graphql
profileInfos(where: {name: {_eq: "Phantom"}}) {
  name
  logo
  tagLine
  descriptionShort
  descriptionLong
  profileType { name }
  profileSector { name }
  profileStatus { name }
  foundingDate
  urls { url urlType { name } }
  root {
    slug
    products {
      name
      description
      productType { name }
      productStatus { name }
      launchDate
    }
    assets {
      name
      ticker
      assetType { name }
      assetStatus { name }
    }
    socials {
      socialType { name }
      name
      socialStatus { name }
    }
  }
}
```

### 2. List All Products of a Type

**VERIFIED WORKING**

```graphql
products(where: {productType: {slug: {_eq: "decentralised_exchange"}}}, limit: 5) {
  name
  description
  productStatus { name slug }
  productType { name slug }
  root { profileInfos { name logo } }
}
```

**Common product type slugs (use these instead of IDs):**

- DEX: `decentralised_exchange`
- Wallet: `wallet`
- Bridge: `bridge`
- Oracle: `oracle`
- RPC Provider: `rpc_provider`
- L1 Blockchain: `l1`
- L2 Blockchain: `l2`
- AI Agent: `ai_agent`
- AI Agent Platform: `ai_agent_platform`

### 3. Find Products Supporting Specific Asset

**VERIFIED WORKING**

```graphql
productAssetRelationships(
  where: {asset: {ticker: {_eq: "USDC"}}}
  limit: 5
) {
  product {
    name
    productType { name }
    root { profileInfos { name } }
  }
  asset {
    name
    ticker
  }
  assetSupportType { name }
}
```

**Support types:**

- Native to (blockchain's native token)
- Supported by (protocol integration)
- Governance (voting token)
- Managed by (issuer)
- Utility (required for use)

### 4. Get Blockchain Deployments

**VERIFIED WORKING**

```graphql
products(
  where: {
    productDeployments: {
      smartContractDeployment: {
        deployedOnProduct: {name: {_eq: "Solana Mainnet"}}
      }
    }
  }
  limit: 5
) {
  name
  productType { name }
  productDeployments {
    smartContractDeployment {
      smartContracts {
        address
      }
      deployedOnProduct { name }
      assetStandard { name }
    }
  }
}
```

### 5. Asset Information with Contracts

**VERIFIED WORKING**

```graphql
assets(where: {ticker: {_eq: "USDC"}}, limit: 1) {
  name
  ticker
  description
  assetType { name }
  assetDeployments {
    smartContractDeployment {
      deployedOnProduct { name }
      assetStandard { name }
      smartContracts {
        address
      }
    }
  }
}
```

### 6. Profile Discovery by Sector

```graphql
profileInfos(
  where: {profileSector: {slug: {_eq: "finance"}}}
  limit: 20
) {
  name
  descriptionShort
  profileSector { name slug }
  root {
    productsAggregate { _count }
    assetsAggregate { _count }
  }
}
```

**Common sector slugs (use these instead of IDs):**

- Finance: `finance`
- Infrastructure: `infrastructure`
- Developer Tooling: `developer_tooling`
- Gaming: `gaming`
- NFTs: `nfts`

### 7. Count Products by Type

**VERIFIED WORKING**

```graphql
productTypes(limit: 10) {
  name
  definition
  productsAggregate {
    _count
  }
}
```

### 8. Social Media Inventory

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

**Social platforms tracked:**

- Twitter/X, Discord, Telegram, GitHub, LinkedIn
- Reddit, YouTube, Instagram, TikTok
- Warpcast, Bluesky, Mastodon, Threads

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
  name
  description
  productType { name slug }
  productStatus { name slug }
  launchDate
}
```

### Cross-Entity Relationships

```graphql
# All L2 blockchains and their native tokens
products(where: {productType: {slug: {_eq: "l2"}}}) {
  name
  description
  productType { name slug }
  root {
    assets(where: {assetTypeId: {_eq: "1"}}) {
      name
      ticker
      description
    }
  }
}
```

### URL-Based Discovery

```graphql
# Find profiles associated with a domain
profileInfos(where: {urls: {url: {_like: "%solana.com%"}}}) {
  name
  urls { url urlType { name } }
}

# Find products associated with a domain
products(where: {urls: {url: {_like: "%solana.com%"}}}) {
  name
  urls { url urlType { name } }
  root { profileInfos { name } }
}
```

### Asset Derivative Chains

```graphql
# All liquid staking tokens (LSTs)
assets(where: {assetType: {slug: {_eq: "liquid_staking_tokens_lsts"}}}) {
  name
  ticker
  description
  assetType { name slug }
}
```

### Ecosystem Analysis — Finding Products on a Chain

When querying a blockchain ecosystem, always start by finding the chain's Product ID
via its Profile. Then use **two complementary methods** to find ecosystem products.

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
# Result: Solana Mainnet → id: "22"
```

**Method A — `supportsProducts` (broader, recommended first)**

Products declare which other products they support. Captures ecosystem membership
even without on-chain deployments (data platforms, trading bots, analytics, etc.).

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

**How `supportsProducts` works:**
- Each product has a `supportsProducts` array of relationships
- `productId` = the product itself, `supportsProductId` = the product it supports
- To find an ecosystem: filter where `supportsProductId` = the chain's product ID

**Method B — `productDeployments` (on-chain only)**

Products with smart contracts deployed on the chain. Narrower — only verified on-chain
deployments, but includes contract addresses.

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

**Counting product types in an ecosystem:**

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

### Asset Ecosystem — Using `productAssetRelationships`

To find which products interact with a specific asset, use `productAssetRelationships`.
This is the asset-centric equivalent of ecosystem queries.

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
- **Native to** — blockchain's native token
- **Supported by** — protocol integration/listing
- **Governance** — voting/governance token
- **Managed by** — token issuer
- **Utility** — required for product usage

### Profile Comparison

```graphql
# Side-by-side comparison
{
  phantom: profileInfos(where: {name: {_eq: "Phantom"}}) {
    name
    root {
      products {
        name
        productType { name slug }
        productStatus { name slug }
      }
    }
  }
  metamask: profileInfos(where: {name: {_eq: "MetaMask"}}) {
    name
    root {
      products {
        name
        productType { name slug }
        productStatus { name slug }
      }
    }
  }
}
```

### Full-Featured Query Example

```graphql
products(
  where: {
    _and: [
      {name: {_like: "%swap%"}},
      {productStatus: {slug: {_eq: "live"}}},
      {launchDate: {_is_null: false}}
    ]
  }
  order_by: [
    { launchDate: Desc }
    { name: Asc }
  ]
  limit: 5
) {
  name
  launchDate
  description
  productType { name slug }
  productStatus { name slug }
  root {
    profileInfos { name }
  }
}
```

---

## Filter Reference

| Filter | GraphQL Syntax | Example |
|--------|---------------|---------|
| Name (exact) | `{name: {_eq: "Jupiter"}}` | Exact match |
| Name (contains) | `{name: {_like: "%wallet%"}}` | Substring search |
| Ticker | `{ticker: {_eq: "SOL"}}` | Asset ticker |
| Product Type (recommended) | `{productType: {slug: {_eq: "decentralised_exchange"}}}` | By type slug |
| Status (recommended) | `{productStatus: {slug: {_eq: "live"}}}` | By status slug |
| Multiple conditions | `{_and: [{...}, {...}]}` | AND conditions |
| Alternative conditions | `{_or: [{...}, {...}]}` | OR conditions |
| Null check | `{field: {_is_null: false}}` | Non-null only |

---

## Troubleshooting

### Query Errors

**"Field not found" errors:**

- Check field names against schema
- Use `profileInfos` not `profiles`
- Use `products` not `product`
- Use nested paths: `root { profileInfos { name } }`

**"Invalid where clause":**

- Ensure proper nesting: `{field: {_operator: value}}`
- Use `_like` with wildcards: `"%value%"`
- `_ilike` is NOT supported, use `_like` instead

**Timeout errors:**

- Reduce query complexity
- Add limits to unbounded queries
- Break large queries into smaller chunks

### Search Not Finding Results

1. **Broaden search**: Use `%partial%` instead of exact match
2. **Check spelling**: "Ethereum" not "Etherium", "Phantom" not "Fantom"
3. **Use correct entity**: Products use `products`, assets use `assets`, profiles use `profileInfos`
4. **Verify status**: Filter by `productStatus: {slug: {_eq: "live"}}` for active products

### Common Pitfalls

- Use `productType` not `type`
- Use `_count` not `count` for aggregates
- Roots don't have `name` — use `slug` or `urlMain`
- Products connect to organizations via `root`, not directly to entities

---

## Known Limitations

1. **`mcp__grid__find`**: Supports `products`, `assets`, and `profiles` lenses — use `mcp__grid__query` for socials and other entities
2. **URL filtering**: Direct filtering by URL content not fully supported — search by name first, then filter URLs in results
3. **Social URL filtering**: Search by social handle name, not by full URL
4. **Case-insensitive search**: `_ilike` is not supported — use `_like` (case-sensitive)

### Workarounds

**For URL-based discovery:**

```graphql
# Search by name first, then check URLs in results
profileInfos(where: {name: {_like: "%keyword%"}}) {
  urls { url urlType { name } }
}
```

**For social media discovery:**

```graphql
# Search by handle name
socials(where: {name: {_like: "%handle%"}}) {
  socialType { name }
  root { profileInfos { name } }
}
```

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

# Get slug values
productTypes { name slug }
assetTypes { name slug }
productStatuses { name slug }

# URL search (workaround)
profileInfos(where: {name: {_like: "%keyword%"}}) {
  urls { url urlType { name } }
}

# Social search by handle
socials(where: {name: {_like: "%handle%"}}) {
  socialType { name slug }
  root { profileInfos { name } }
}

# Multi-criteria with slugs
_and: [
  {productType: {slug: {_eq: "wallet"}}},
  {productStatus: {slug: {_eq: "live"}}}
]
_or: [
  {productType: {slug: {_eq: "l1"}}},
  {productType: {slug: {_eq: "l2"}}}
]

# Pattern matching
_like: "%value%" (contains)
_eq: "exact" (exact match)
```
