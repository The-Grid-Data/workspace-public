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

**TGS — The Grid Schema** is the data structure for querying The Grid's
API. This skill is the operating guide: the patterns and field reference
for searching that schema via the MCP tools below.

TGS is read by both humans (Discovery, SEO) and machines (MCP clients,
agents, partner integrations). Every record has to serve both — the
schema is *queried, not curated*: types, statuses, and relationships are
load-bearing so that queries return correct answers without per-row
babysitting.

The Grid contains structured data on 3,000+ Web3 profiles (companies,
protocols, DAOs), 6,400+ products (DEXs, wallets, bridges), 1,600+ assets
(tokens, coins, stablecoins), and the relationships between them.

**Further reading.** Local references live in
[references/grid-mcp-guide.md](references/grid-mcp-guide.md) (the long-form
query reference) and `public/references/` (`tgs-field-descriptions.md`,
`tgs-json-structure.md`). The public ER diagram is at
[dbdiagram.io/d/TGS10-PUBLIC](https://dbdiagram.io/d/TGS10-PUBLIC-68c982ab1ff9c616bdf66b03).
The canonical TGS narrative (team access) is on Slite under
*TGS Data Model* and *Lenses*.

## Vocabulary

Short glosses for the terms the Concepts section relies on. Each entry
points at the source of truth for the deep version.

- **Lens.** *Different perspectives or filters through which you can view
  Web3 data, similar to how a camera lens focuses on specific subjects.*
  The 11 canonical lenses are **Profiles** (organisations, protocols,
  DAOs), **Products** (their offerings — DEX, wallet, bridge, L1/L2),
  **Assets** (tokens, coins, stablecoins), **Entities** (legal structures
  behind a profile — foundation, corporation), **Smart Contract
  Deployments** (on-chain footprint — contract addresses, networks,
  standards), **Socials**, **URLs**, **Media**, **Attributes** (typed
  key-value extras, narrowly scoped), **Internal Helpers** (system
  wiring, incl. the `roots` and `gridRank` tables), and **Geography**.
  See the lens table in
  [references/grid-mcp-guide.md](references/grid-mcp-guide.md) (lines
  99–111). The Slite *Lenses* page has a longer walkthrough.

- **Profile vs `profileInfos`.** `profiles` is the lens (the query area);
  `profileInfos` is the table inside it that holds name, descriptions,
  type, status, sector — the per-profile metadata. **Always query
  `profileInfos`, not `profiles` directly.** That's why every example in
  this skill opens with `profileInfos(where: ...)`.

- **Root.** *Top-level profile records — the canonical identity anchor
  for every profile in the Grid. All other data hangs off a root.*
  (See `public/references/tgs-field-descriptions.md`.) You traverse via
  `.root` from `profileInfos`; there is no top-level `rootId` column.

- **Slug.** Human-readable, stable enum identifier (e.g.
  `decentralised_exchange`). Filter on slugs, not numeric IDs — slugs
  survive ID churn and are self-documenting.

- **Classification layer.** The collective name for the `*Types`,
  `*Statuses`, and `*Tags` enum tables that constrain each lens
  (`profileTypes`, `productStatuses`, `assetSupportTypes`, `tags`, etc.).
  Every primary lens has at least one. They're the load-bearing handle
  for query-driven correctness: when a query returns the wrong rows, the
  fix is almost always at the classification layer, not the content.

- **Cluster.** *Local term in this skill* (not in canonical TGS docs).
  We use **cluster** for *the root plus everything that hangs off it* —
  `products`, `assets`, `entities`, `socials`, `media`, `urls`, plus
  `rootRelationships` in both directions. It's the unit of data you need
  to read together to answer most questions about a real-world entity.

- **Graph.** The directed connections that link clusters together:
  `rootRelationships` between roots, `supportsProducts` between products,
  `productAssetRelationships`, and the deployment chains
  (`productDeployments` / `assetDeployments` → `smartContractDeployments`
  → `smartContracts`).

## Concepts

Read these before querying or writing. The patterns below assume them.
Each concept is stated as: **what it is → why it's true → how it fails →
what to do differently**. Knowing *why* lets you handle cases the patterns
don't anticipate.

### 1. Two consumers, one schema

The intro stated the principle; this is its operational consequence.
Discovery, SEO crawlers, MCP clients, and partner integrations
(Tether, Solana pay.sh, Stablecoin Standard, GridTree) all read the
same records — every field has to serve both eyes and machines.

*Failure mode:* a fix that improves how a profile renders in the UI but
breaks the field shape a partner integration depends on, or vice versa.

*Action:* when changing data or classifications, check both surfaces.
A "small fix" to a name, status, or relationship is a write to a
machine-readable contract, not just a page edit.

### 2. Queried, not curated

The schema is meant to be self-consistent so queries return correct answers
without per-row babysitting. If you find yourself reasoning "this one is
special", the schema is wrong, not the row.

*Failure mode:* reaching for the `attributes` lens to encode something
that should be a structural field. Attributes drift; structural fields
don't.

*Action:* before writing a workaround, ask whether the right answer is a
new value in the classification layer (a new type, status, or
relationship) rather than freeform metadata. Use `attributes` only for
genuinely freeform data that has no place in fixed fields.

### 3. A profile is a record, not a page

One real-world entity = one **root** (see Vocabulary). The root is the
load-bearing identifier; `profileInfos` is the table that carries the
human-readable metadata under it; URLs, socials, media all *derive from*
the root rather than duplicating it. This is also why every query in
this skill opens with `profileInfos(where: ...)` — that's the entry door
into a root.

*Failure mode:* same entity appears under two roots, two slugs, or two
URLs — page authority splits, agent answers diverge, the same fix has to
land twice.

*Action:* before creating a new profile, search `profileInfos` for the
name and check `rootRelationships` for the entity under another root. Two
records for the same entity is the bug.

### 4. The unit of meaning is the cluster

A profile alone is incomplete. The **cluster** (see Vocabulary) is the
unit you need to read together — the root plus everything that hangs
off it. Read all of them before drawing a conclusion.

*Failure mode:* reporting "Tether is missing entities" when the
foundation is a linked profile via `relationshipType: "Managed by"`.
Reporting "Tether has no USDT0" when USDT0 is a child profile with its
own `assets`. Audits that re-find the same gaps every time.

*Action:* when auditing or answering "what does X have", fetch the whole
cluster. The pattern below binds `$rootId` to the root's `id` (resolved
by an outer query or inlined as a literal — there is no top-level
`rootId` column to filter on):

```graphql
{
  profileInfos(where: {name: {_eq: "Tether"}}) {
    name root {
      id
      entities { name entityType { name } country }
      products { name productType { name } productStatus { name } }
      assets { name ticker assetType { name } assetStatus { name } }
      parentRootRelationships: rootRelationships(where: {childRootId: {_eq: $rootId}}) {
        relationshipType { name }
        parentRoot { profileInfos { name } }
      }
      childRootRelationships: rootRelationships(where: {parentRootId: {_eq: $rootId}}) {
        relationshipType { name }
        childRoot { profileInfos { name } products { name } assets { name } }
      }
    }
  }
}
```

If a question can be answered without traversing the cluster, you're
probably answering the wrong question.

### 5. Classification before content

Type, status, and relationship fields are load-bearing; descriptions and
URLs are downstream of them (see Vocabulary → Classification layer for
the table inventory).

*Failure mode:* a profile classified as "Network" whose description and
products read like a "Company". A product typed `decentralised_exchange`
whose `productAssetRelationships` look like a lending protocol. Mismatch
between type and content is the bug, and it's the type that's usually
wrong.

*Action:* when reading, trust the classification layer over the
description. When writing, set type/status/relationships first, then
fill content to match. Use `get-tgs(action: "enum", name: "...")` to
confirm valid values before guessing.

### 6. Status is a claim about now

Status (one `*Statuses` table per primary lens — profile, product, asset,
social) is a claim about the *current* state of the entity, not a
default. "Live" must be verifiable today. "Discontinued" or "Sunset"
must reflect that the thing actually stopped. "Not set" is never an
acceptable resting state.

*Failure mode:* a product sunset months ago that still reads `Live`. A
profile with status `not set` because nobody verified. Stale status is a
quiet bug — queries return wrong results without any error.

*Action:* when writing a status, name the verification source (URL, dated
note, social post). When reading, treat any `Live` older than ~90 days
without a recent update as a candidate for re-verification. When in doubt
about `not set`, set it correctly before saving.

### 7. The graph IS the product

The **graph** (see Vocabulary) isn't metadata bolted on the side — it's
the answer to most interesting questions. `rootRelationships` link
profiles to profiles; `supportsProducts` and `productAssetRelationships`
link the typed relationship lenses; the deployment chains
(`productDeployments` / `assetDeployments` → `smartContractDeployments`)
link on-chain footprint to the rest.

*Failure mode:* an orphan. A subsidiary profile with no link to its
parent. An asset issued by a profile but not connected to it. A product
that supports a chain but isn't in `supportsProducts`. Orphans break
every consumer that traverses the graph (which is most of them).

*Action:* when you create or fix an entity, ask "what else does this
connect to?" and write the relationship. When you read and the answer
looks empty, check whether the answer is actually one hop away in the
graph.

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

Reading from the **root** outward — this is the *cluster* the Vocabulary
defines. Lenses that hang directly off the root are first-class
children; enum tables (tags, types, statuses) attach via join tables and
appear at the bottom.

```
Root
  |-- profileInfos (name, descriptions, type, status, sector)
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
  |-- Media (Icons, Logos, Headers)
  |-- URLs (Website, Docs, App, Blog, etc.)
  |-- Attributes (Bluechip ratings, ChainIDs — narrow key-value extras)
  |-- gridRank (importance score)
  +-- rootRelationships (parent/child, acquisitions — links to other roots)

profileTags  ─join→  tags          (event, community, tech, paradigm…)
```

**Key relationships (the graph):**
- `root` is the anchor — every cluster member traces back to it.
- `supportsProducts` links products to products (ecosystem membership).
- `productAssetRelationships` links products to assets.
- `rootRelationships` links roots to roots (parent/child, acquisitions).
- `profileTags` links roots to entries in the `tags` enum table —
  tags are *not* a peer lens; they attach via this join.

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
