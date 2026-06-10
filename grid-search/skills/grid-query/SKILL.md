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

The Grid contains structured data on thousands of Web3 profiles (companies,
protocols, DAOs), products (DEXs, wallets, bridges), and assets (tokens,
coins, stablecoins), plus the relationships between them. For live totals,
count directly — e.g. `profileInfosAggregate { _count }` — rather than
quoting a number that goes stale.

**Further reading.** The long-form query reference is bundled with this skill at
[references/grid-mcp-guide.md](references/grid-mcp-guide.md). For field-level
detail and enum values, query the live schema directly with
`get-tgs(action: "schema", name: "...")` and `get-tgs(action: "enum", name: "...")`
— that is the source of truth, not any static file. The public ER diagram is at
[dbdiagram.io/d/TGS10-PUBLIC](https://dbdiagram.io/d/TGS10-PUBLIC-68c982ab1ff9c616bdf66b03).
The canonical TGS narrative (team access) is on Slite under
*TGS Data Model* and *Lenses*.

## Lenses — one entity, many views

A **lens** is one perspective on a single Web3 entity (a company, protocol,
DAO, network, token issuer, …). The same entity carries many lenses at once;
each answers a different class of question. You don't query "a row" — you pick
the lens that matches your question. Every lens hangs off the same `root` (see
*Concepts → the unit of meaning is the cluster*), so one entity resolves
consistently across all of them.

> *Analogy (intuition only — the data is Web3 entities, not retail):* if a
> profile were a physical shop, each lens is a different question you'd ask
> about it — what's the sign say, what's for sale, how do I pay, is it
> registered, where are its channels. Don't take "shop" literally; most
> profiles are protocols, networks, or issuers.

Run `get-tgs(action: "lenses")` for the authoritative current set. The everyday
lenses:

### Profile lens — the high-level overview
- **Reveals:** name, profile type, sector, and what the profile does.
- **Use when:** you need a fast "what is this" before digging deeper.
- **Query:** `profileInfos` → `name`, `profileType`, `profileSector`,
  `profileStatus`, `descriptionShort`.

### Product lens — the offerings
- **Reveals:** what each product does, how it describes itself, what it works
  with (`supportsProducts`), launch date and status.
- **Use when:** you need detail on specific tools or services.
- **Query:** `products` → `productType`, `productStatus`, `launchDate`,
  `supportsProducts`.

### Asset lens — tokens & currencies
- **Reveals:** the profile's own tokens, token type and technical details,
  network compatibility (via deployments). What products *accept or support*
  lives one hop over in **Product-Asset Relationships**
  (`productAssetRelationships` → `assetSupportType`, e.g. *Payment Accepted by*).
- **Use when:** you're researching tokens or checking payment/support.
- **Query:** `assets` → `ticker`, `assetType`, `assetStatus`, `assetDeployments`.

### Entity lens — legal structure
- **Reveals:** legal registration, corporate structure, licences/identifiers
  (LEI, local registration), and geographic presence (country of incorporation).
- **Use when:** you need to verify legitimacy or do due diligence.
- **Query:** `entities` → `entityType`, `countryId`, `leiNumber`,
  `localRegistrationNumber`.

### Socials lens — community channels
- **Reveals:** official accounts (Twitter/X, Discord, Telegram, GitHub),
  account status (Active/Inactive), and platform type.
- **Use when:** you want official channels or to assess community presence.
- **Query:** `socials` → `socialType`, `name`, `socialStatus`.

### URLs lens — web presence
- **Reveals:** primary websites, documentation/whitepapers, web apps, blogs,
  support pages — each typed.
- **Use when:** you need direct links to resources.
- **Query:** `urls` → `url`, `urlType`.

### Smart Contract Deployments lens — on-chain footprint
- **Reveals:** contract addresses, the networks deployed on, token standards
  (ERC-20, SPL, …), and deployment history.
- **Use when:** you need technical on-chain information.
- **Query:** `smartContractDeployments` → `smartContracts { address }`,
  `assetStandard`, `deployedOnProduct`; reached via a product's
  `productDeployments` or an asset's `assetDeployments`.

**Three more lenses complete the set:** **Media** (logos, icons, headers),
**Attributes** (narrow typed extras like ratings or ChainIDs), and **Internal
Helpers** (the system wiring — `roots`, `gridRank`, `rootRelationships`).

## Vocabulary

Short glosses for the terms the Concepts section relies on. Each entry
points at the source of truth for the deep version.

- **Lens.** A perspective on a single real-world entity — the camera angle
  you choose to answer a particular question. See the **Lenses — one entity,
  many views** section above for the full per-lens walkthrough. Note:
  "Geography" is *not* a lens — it's a `tagTypes` value plus the `countries`
  enum. **Run `get-tgs(action: "lenses")` for the authoritative, current
  set** rather than trusting any hand-written list.

- **Profile vs `profileInfos`.** `profiles` is the lens (the query area);
  `profileInfos` is the table inside it that holds name, descriptions,
  type, status, sector — the per-profile metadata. **Always query
  `profileInfos`, not `profiles` directly.** That's why every example in
  this skill opens with `profileInfos(where: ...)`.

- **Root.** *Top-level profile records — the canonical identity anchor
  for every profile in the Grid. All other data hangs off a root.* You
  traverse via `.root` from `profileInfos`; there is no top-level `rootId`
  column.

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
get-tgs(action: "enum", name: "productTypes") # List every product type (const/title/description)
get-tgs(action: "schema", name: "profiles")   # Get full profile schema
```

**Always use `get-tgs` before guessing at field names or enum values.** It is the source of truth.

> **Don't introspect.** If you're about to send a GraphQL `__schema` or
> `__type` introspection query through `mcp__grid__query`, stop —
> `get-tgs(action: "schema", name: "...")` and `get-tgs(action: "lenses")`
> are purpose-built for exactly that and return the source-of-truth shape.
> Introspection is slower, noisier, and easy to get wrong.

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

**Count matches — use `<entity>Aggregate { _count }`:**
```graphql
# Total profiles
profileInfosAggregate { _count }

# Filtered count — wrap the filter in `filter_input`
productsAggregate(filter_input: {where: {productStatus: {slug: {_eq: "live"}}}}) {
  _count
}
```
This API does **not** use the Hasura shape `entity_aggregate { aggregate { count } }`
— that field does not exist and the query will fail. It's `entityAggregate { _count }`.

## Tag Queries

Tags are a powerful way to find profiles by event attendance, ecosystem membership,
community affiliation, or technology focus. Tags are grouped into tag types.

**Tag types (illustrative — run `get-tgs(action: "enum", name: "tagTypes")` for the
current set):** Event, Community, Tech, Paradigm, Geography, Hackathon, Report

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

**Relationship types** (run `get-tgs(action: "enum", name: "rootRelationshipTypes")` for the current set): Acquired by, Associated with, Managed by, Product of

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
there are **two ways** to find products on that chain. Filter by the chain product's
**name**, never a hardcoded numeric id — ids are not stable (newer rows get hash ids
like `id1739677005-…`), so a baked-in `"22"` rots silently.

If you do need the chain's product id (e.g. to reuse across several queries), resolve
it live first, then bind it — don't memorise it:

```graphql
profileInfos(where: {name: {_eq: "Solana"}}) {
  name
  root {
    products(where: {productType: {slug: {_in: ["l1", "l2"]}}}) { id name }
  }
}
```

**Method A — `supportsProducts` (broader, recommended first)**

Products declare which other products they support. This captures ecosystem membership
even without on-chain deployments (e.g., data platforms, trading bots, analytics tools).
Filter through the join to the supported product's name:

```graphql
products(
  where: {
    supportsProducts: {
      supportsProduct: {name: {_eq: "Solana Mainnet"}}
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

**`assetSupportType` values** (run `get-tgs(action: "enum", name: "assetSupportTypes")` for the current set): Native to, Supported by, Governance, Managed by,
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

**Media types** (run `get-tgs(action: "enum", name: "mediaTypes")` for the current set): Icon, Logo on white, Logo on black, Header

## Grid Rank

Each profile carries an importance measure based on ecosystem connectivity —
how many other profiles, products, and assets it links to. Two fields:

- **`score`** — the raw connectivity magnitude. Higher is better; unbounded
  (e.g. Ethereum ≈ 4,799, a long-tail profile in the low tens).
- **`ranking`** — the profile's **ordinal position** on the global leaderboard.
  **`1` is the most important profile** (Ethereum is #1, Solana #2, Bitcoin #3).
  Ascending `ranking` corresponds to descending `score`.

Prefer `ranking` for "is this a top-N profile / where does it sit" and `score`
for the raw magnitude or for comparing two profiles' relative weight.

```graphql
profileInfos(where: {name: {_eq: "Uniswap"}}) {
  name
  root {
    gridRank { score ranking }   # e.g. { score: 292, ranking: 17 }
  }
}
```

**Leaderboard — top profiles by rank** (sort `ranking` ascending):

```graphql
gridRank(order_by: {ranking: Asc}, limit: 10) {
  ranking score
  root { profileInfos { name } }
}
```

> Note: `ranking` is a live field but is not yet listed in
> `get-tgs(action: "schema", name: "gridRank")` (which currently documents only
> `score`) — query it anyway; it resolves.

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
