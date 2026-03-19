# TGS JSON Output Structure Reference

**Updated 2026-02-17 — Nested structure confirmed via validated import.**

This file defines the exact structure for TGS JSON output. **Read this before generating
any JSON.** The structure uses **nested arrays** — URLs, socials, media, relationships,
and deployments are nested inside their parent objects, not in separate top-level arrays.

For detailed field descriptions with full context, see `tgs-field-descriptions.md`.

## Key Rules

1. **Nested structure** — URLs, socials, media nest inside `profileInfos`, `products`,
   `assets`, and `entities`. Asset deployments nest inside `assets` with a
   `smartContractDeployment` wrapper. Relationship arrays nest inside `products`.
2. **Every field value is a string** — including booleans (`"1"` / `"2"`) and numbers
3. **Enum IDs come from `tgs-enums.md`** — never invent them
4. **No `rootId` field** — the import system does not use rootId
5. **Cross-references** use Grid IDs for existing entities. For new entities created in
   the same JSON, use `[NEW:arrayName.index]` (see Cross-Referencing below)
6. **`feedback`** — required root-level string summarizing the enrichment

## Top-Level Structure

```json
{
  "profileInfos": [ ... ],
  "products": [ ... ],
  "assets": [ ... ],
  "entities": [ ... ],
  "feedback": "Summary of the enrichment research..."
}
```

Only four data arrays plus a `feedback` string at the top level. Everything else nests.

---

## Field Reference

Fields marked `[enum]` must use IDs from `tgs-enums.md`. Fields marked `[ref]` reference
another entity by its Grid ID. Leave fields blank (`""`) when data is not available.

### profileInfos

Each profile contains its own `urls`, `socials`, and `media` arrays.

| Field | Type | Notes |
|-------|------|-------|
| `name` | string | Full official name as used in branding |
| `profileTypeId` | string [enum] | References `profileTypes` enum. DAO, Company, Project, etc. |
| `profileSectorId` | string [enum] | References `profileSectors` enum. Finance, Gaming, Infrastructure, etc. |
| `profileStatusId` | string [enum] | References `profileStatuses` enum. Active, Inactive, Closed, etc. |
| `foundingDate` | string | ISO 8601: YYYY-MM-DD or YYYY-MM or YYYY |
| `descriptionShort` | string | Brief objective overview. Hard limit 200 chars. No subjective claims |
| `descriptionLong` | string | Detailed overview: mission, audience, features. Hard limit 500 chars. Objective |
| `descriptionMarketing` | string | Promotional — persuasive language permitted. Leave `""` if none |
| `tagLine` | string | Brief slogan. Normalize to single line, single spaced, sentence case |
| `urls` | array | Nested URL objects (see urls below) |
| `socials` | array | Nested social objects (see socials below) |
| `media` | array | Nested media objects (see media below) |

**NOT in profileInfos:** `rootId`, `logo`, `icon` — use `media` array for branding images.

### products

Each product contains its own `urls`, `socials`, `media`, `productAssetRelationships`,
`productDeployments`, and `supportsProducts` arrays.

| Field | Type | Notes |
|-------|------|-------|
| `name` | string | Official product name as used in branding |
| `productTypeId` | string [enum] | References `productTypes` enum. See product-type-definitions.md |
| `description` | string | Hard limit 500 chars |
| `productStatusId` | string [enum] | References `productStatuses` enum. Live, In Development, etc. |
| `isMainProduct` | string [enum] | `"1"` = True, `"2"` = False. Only ONE product per profile can be `"1"` |
| `launchDate` | string | ISO 8601: YYYY-MM-DD or YYYY-MM |
| `urls` | array | Product-specific URLs (use urlTypeId `"6"` for Product) |
| `productDeployments` | array | Nested deployment references (usually empty `[]`) |
| `productAssetRelationships` | array | Nested asset relationship objects (see below) |
| `supportsProducts` | array | Nested product support links (see below) |
| `socials` | array | Product-specific socials (usually empty `[]`) |
| `media` | array | Product-specific media (usually empty `[]`) |

### assets

Each asset contains nested `urls`, `assetDeployments`, `socials`, `media`, and `metadata`.

| Field | Type | Notes |
|-------|------|-------|
| `name` | string | Official asset name as used in branding and trading materials |
| `ticker` | string | Capital letters and numbers only. ALL CAPS trading symbol |
| `icon` | string (URL) | Square icon for the asset (blank if unavailable) |
| `description` | string | 3 sentences (Identity, Usage, Hook). Start with TICKER in ALL CAPS. Target 280–350 chars, hard limit 500. See asset-description-guidelines.md |
| `assetTypeId` | string [enum] | References `assetTypes` enum. Utility, Governance, Currency, Stablecoin, etc. |
| `assetStatusId` | string [enum] | References `assetStatuses` enum. Active, Inactive, In Development |
| `urls` | array | Asset-specific URLs (tokenomics, explorer pages). Empty `[]` if none |
| `assetDeployments` | array | Nested deployment objects (see below). Empty `[]` if no contracts |
| `socials` | array | Asset-specific socials (usually empty `[]`) |
| `media` | array | Asset-specific media (usually empty `[]`) |
| `metadata` | object | Sources and reasoning (see metadata below) |

**NOT in assets:** `rootId`, `address` — contract addresses go inside `smartContracts`.
**WRONG field name:** `smartContractDeployments` does NOT exist on assets. Use `assetDeployments`.

### entities

Each entity contains nested `urls` and `socials` arrays.

| Field | Type | Notes |
|-------|------|-------|
| `name` | string | Legal name as registered with regulatory bodies |
| `tradeName` | string | Additional trading names, brands, or DBAs |
| `entityTypeId` | string [enum] | References `entityTypes` enum. Corporation, Foundation, Startup, etc. |
| `dateOfIncorporation` | string | ISO 8601: YYYY-MM-DD |
| `localRegistrationNumber` | string | Government registration number (blank if unavailable) |
| `taxIdentificationNumber` | string | Tax ID (blank if unavailable) |
| `address` | string | Full registered physical address: street, city, state, postal code |
| `countryId` | string [enum] | References `countries` enum. ISO 3166-1 alpha-2 codes |
| `leiNumber` | string | 20-char LEI from GLEIF (blank if unavailable) |
| `urls` | array | Entity-specific URLs (company website). See urls below |
| `socials` | array | Entity-specific socials (usually empty `[]`) |
| `metadata` | object | Sources and reasoning (see metadata below) |

---

## Nested Object Reference

These objects appear **inside** their parent arrays, never as top-level arrays.

### urls (nested in profileInfos, products, assets, and entities)

| Field | Type | Notes |
|-------|------|-------|
| `urlTypeId` | string [enum] | References `urlTypes` enum. Main, Documentation, Blog, Whitepaper, Product, etc. |
| `url` | string (URL) | Must start with `http://` or `https://` |

**No `rootId` field.** Profile-level URLs go in `profileInfos[].urls`. Product-specific
URLs (app links, product pages) go in `products[].urls` with urlTypeId `"6"` (Product).
Asset-specific URLs (token pages, explorers) go in `assets[].urls`.
Entity URLs (company website) go in `entities[].urls`.

**URL type constraints by parent object** — not all URL types are valid in all contexts:

| Parent | Valid URL Types | Common Examples |
|--------|----------------|-----------------|
| `profileInfos` | Main (2), Documentation (4), Blog (3), Whitepaper (5), Terms (`id1756948093...`), Privacy (`id1756948673...`), Media Kit (`id1751290760...`), Apple App Store (`id1746031992...`), Google Play Store (`id1747724202...`), Governance (`id1746429910...`), Forum (`id1746120153...`), Extra source (`id1737127021...`) | Homepage, docs site, legal pages |
| `products` | Product (6), Documentation (4), Extra source (`id1737127021...`) | App access URL, product-specific docs |
| `assets` | Asset Info (`id1760009128...`), Extra source (`id1737127021...`), Deployment (`id1757498657...`) | Token landing page, block explorer |
| `entities` | Entity (7) | Company website |

**Common mistakes:**
- ❌ `assets[]` using Product (6) — asset-specific project pages use Asset Info (`id1760009128...`)
- ❌ Block explorer links (Etherscan, Basescan, etc.) using Asset Info — use Extra source (`id1737127021...`)
- ✅ Token product/landing page (e.g. `midas.app/mtbill`, `protocol.io/token`) → Asset Info
- ✅ `https://etherscan.io/token/0x...` or any block explorer URL → Extra source

### socials (nested in profileInfos, products, assets, and entities)

| Field | Type | Notes |
|-------|------|-------|
| `socialTypeId` | string [enum] | References `socialTypes` enum. Twitter/X, Discord, Telegram, etc. |
| `socialStatusId` | string [enum] | References `socialStatuses` enum. Active, Inactive, etc. |
| `name` | string | Handle from URL path, NOT display name |
| `url` | string (URL) | Full URL to the social profile |

**New vs old:** Socials now include a full `url` field in addition to the `name` handle.
Profile-level socials go in `profileInfos[].socials`. Product-specific socials (rare) go
in `products[].socials`. Most products have `"socials": []`.

### media (nested in profileInfos and products)

| Field | Type | Notes |
|-------|------|-------|
| `url` | string (URL) | Image file URL |
| `mediaTypeId` | string [enum] | References `mediaTypes` enum. Logo on white, Logo on black, Icon, Header |

**No `rootId` field.** Profile logos/icons go in `profileInfos[].media`. Product-specific
branding (rare) goes in `products[].media`. Most products have `"media": []`.

### assetDeployments (nested in assets)

Each entry in `assetDeployments` wraps its data in a `smartContractDeployment` object.
Reference fields (`deploymentType`, `deployedOnProduct`) use **nested ID objects**,
not flat strings.

**Structure of each assetDeployments entry:**

```json
{
  "smartContractDeployment": {
    "deploymentType": { "id": "3" },
    "deployedOnProduct": { "id": "21" },
    "assetStandard": { "id": "1" },
    "smartContracts": [ ... ],
    "urls": [],
    "metadata": { "sources": [...], "reasoning": "..." }
  }
}
```

| Field (inside `smartContractDeployment`) | Type | Notes |
|-------|------|-------|
| `deploymentType` | object | `{ "id": "..." }` — enum from tgs-enums.md. Use `"3"` (Mint) for token deployments |
| `deployedOnProduct` | object | `{ "id": "..." }` — Grid ID of the blockchain (e.g. `"21"` for Ethereum Mainnet) |
| `assetStandard` | object | `{ "id": "..." }` — ERC-20, SPL, BEP-20, etc. Nested object, not flat string |
| `smartContracts` | array | Nested contract address objects (see below) |
| `urls` | array | Deployment-specific URLs (usually empty `[]`) |
| `metadata` | object | Sources and reasoning for this deployment |

**Deployment type for tokens:** Use `"3"` (Mint) for asset token deployments. `"5"` (General)
is for product/protocol deployments, not token mints.

**WRONG field names to avoid:**
- `deploymentTypeId` — use `deploymentType: { "id": "..." }` instead
- `deployedOnId` — use `deployedOnProduct: { "id": "..." }` instead
- `assetStandardId` — use `assetStandard: { "id": "..." }` instead
- `smartContractDeployments` — the field on assets is `assetDeployments`

### smartContracts (nested in smartContractDeployment)

| Field | Type | Notes |
|-------|------|-------|
| `name` | string | Contract name if known, blank otherwise |
| `address` | string | Blockchain address |
| `deploymentDate` | string | ISO 8601: YYYY-MM-DD |
| `metadata` | object | Sources and reasoning for this contract address |

**No `deploymentId` field** — the parent relationship is implicit from nesting.

### metadata (on products, assets, entities, deployments, and contracts)

Optional but recommended for traceability. Documents where data came from.

| Field | Type | Notes |
|-------|------|-------|
| `sources` | array | Array of `{ "url": "...", "snippet": "..." }` objects |
| `reasoning` | string | Explanation of classification or verification decisions |

Include on all non-trivial objects (products, assets, entities, deployments, contracts).

### productAssetRelationships (nested in products)

| Field | Type | Notes |
|-------|------|-------|
| `assetId` | string [ref] | Grid ID or `[NEW:assets.N]` reference |
| `assetSupportTypeId` | string [enum] | Native to, Supported by, Utility, Payment Accepted by, etc. |

**No `productId` field** — the parent product is implicit from nesting.

### productDeployments (nested in products)

| Field | Type | Notes |
|-------|------|-------|
| `deploymentId` | string [ref] | References a deployment |

Usually empty (`[]`). Only populated when a product has its own on-chain deployment
separate from asset deployments.

### supportsProducts (nested in products)

| Field | Type | Notes |
|-------|------|-------|
| `supportsProductId` | string [ref] | Grid ID of the product being supported |

**No `productId` field** — the parent product is implicit from nesting.

---

## Cross-Referencing

For **existing Grid entities**, use their Grid ID directly (e.g. `"21"` for Ethereum
Mainnet, `"24"` for Bitcoin).

For **new entities created within the same JSON**, use this convention:

| Reference Field | Points To | Example |
|----------------|-----------|---------|
| `productAssetRelationships[].assetId` | `assets[N]` | `"[NEW:assets.0]"` |
| `supportsProducts[].supportsProductId` | Grid product | Use Grid ID directly |

The `N` is the 0-based index into the top-level array. `[NEW:assets.0]` = first asset,
`[NEW:assets.1]` = second asset.

For entities **not in Grid and not in this JSON**: `"[NOT_IN_GRID: Entity Name]"`.

---

## feedback Field

A root-level string summarizing the enrichment research. Include: profile type, key
products, asset count, notable findings. Example:

```json
{
  "feedback": "ExampleDEX is a DeFi project with 1 product (DEX on Ethereum), 1 governance token (EXD, ERC-20). Platform launched September 2023. Entity incorporated in Cayman Islands."
}
```

---

## Complete Example

A DEX with one product, one governance token, and one ERC-20 deployment on Ethereum:

```json
{
  "profileInfos": [
    {
      "name": "ExampleDEX",
      "profileTypeId": "14",
      "profileSectorId": "1",
      "profileStatusId": "2",
      "foundingDate": "2023-06",
      "descriptionShort": "ExampleDEX is a DEX that enables token swaps on Ethereum with automated liquidity pools.",
      "descriptionLong": "ExampleDEX is a DEX on Ethereum that enables token swaps through automated liquidity pools. Users can trade tokens, provide liquidity to earn fees, and participate in governance through the EXD token. The protocol uses a constant-product formula and supports permissionless pool creation for any ERC-20 token pair.",
      "descriptionMarketing": "",
      "tagLine": "Trade any token on Ethereum",
      "urls": [
        { "urlTypeId": "2", "url": "https://exampledex.io" },
        { "urlTypeId": "4", "url": "https://docs.exampledex.io" },
        { "urlTypeId": "3", "url": "https://exampledex.io/blog" },
        { "urlTypeId": "5", "url": "https://exampledex.io/whitepaper-v2.pdf" },
        { "urlTypeId": "id1756948093-KGS62Bg6RYizSQJ-Hq2_RA", "url": "https://exampledex.io/terms" },
        { "urlTypeId": "id1756948673-zIXSESaWSJS7Wem9Vr1fKQ", "url": "https://exampledex.io/privacy" }
      ],
      "socials": [
        {
          "socialTypeId": "1",
          "socialStatusId": "id1758208503-WqsLQN06TLWVg0D7tkU4IQ",
          "name": "exampledex",
          "url": "https://twitter.com/exampledex"
        },
        {
          "socialTypeId": "4",
          "socialStatusId": "id1758208503-WqsLQN06TLWVg0D7tkU4IQ",
          "name": "exampledex",
          "url": "https://discord.gg/exampledex"
        },
        {
          "socialTypeId": "4157",
          "socialStatusId": "id1758208503-WqsLQN06TLWVg0D7tkU4IQ",
          "name": "exampledex",
          "url": "https://www.tiktok.com/@exampledex"
        }
      ],
      "media": [
        {
          "url": "https://exampledex.io/logo.svg",
          "mediaTypeId": "id1756257129-Wn3d1nkJTKuDH_j3SpS2_A"
        }
      ]
    }
  ],
  "products": [
    {
      "name": "ExampleDEX",
      "productTypeId": "25",
      "description": "ExampleDEX is a DEX that uses automated liquidity pools for ERC-20 token swaps on Ethereum. The protocol supports permissionless pool creation and features concentrated liquidity ranges for capital-efficient trading.",
      "productStatusId": "5",
      "isMainProduct": "1",
      "launchDate": "2023-09-15",
      "urls": [
        { "urlTypeId": "6", "url": "https://app.exampledex.io" }
      ],
      "productDeployments": [],
      "productAssetRelationships": [
        {
          "assetId": "[NEW:assets.0]",
          "assetSupportTypeId": "5"
        }
      ],
      "supportsProducts": [],
      "socials": [],
      "media": []
    }
  ],
  "assets": [
    {
      "name": "ExampleDEX",
      "ticker": "EXD",
      "icon": "https://exampledex.io/exd-icon.png",
      "description": "EXD is the governance token of ExampleDEX. It is used for voting on protocol proposals, fee structure changes, and treasury allocations. EXD holders earn a share of trading fees through a staking mechanism.",
      "assetTypeId": "7",
      "assetStatusId": "2",
      "urls": [],
      "assetDeployments": [
        {
          "smartContractDeployment": {
            "deploymentType": { "id": "3" },
            "deployedOnProduct": { "id": "21" },
            "assetStandard": { "id": "1" },
            "smartContracts": [
              {
                "name": "EXD Token",
                "address": "0x1234567890abcdef1234567890abcdef12345678",
                "deploymentDate": "2023-09-15",
                "metadata": {
                  "sources": [
                    {
                      "url": "https://etherscan.io/token/0x1234567890abcdef1234567890abcdef12345678",
                      "snippet": "EXD token contract on Ethereum Mainnet."
                    }
                  ],
                  "reasoning": "Contract address confirmed via Etherscan."
                }
              }
            ],
            "urls": [],
            "metadata": {
              "sources": [
                {
                  "url": "https://docs.exampledex.io/tokenomics",
                  "snippet": "EXD is an ERC-20 governance token deployed on Ethereum."
                }
              ],
              "reasoning": "ERC-20 on Ethereum Mainnet confirmed via official docs."
            }
          }
        }
      ],
      "socials": [],
      "media": [],
      "metadata": {
        "sources": [
          {
            "url": "https://docs.exampledex.io/tokenomics",
            "snippet": "EXD governance token for voting and fee distribution."
          }
        ],
        "reasoning": "Governance token confirmed via official documentation."
      }
    }
  ],
  "entities": [
    {
      "name": "ExampleDEX Labs Inc.",
      "tradeName": "ExampleDEX",
      "entityTypeId": "3",
      "dateOfIncorporation": "2023-03-15",
      "localRegistrationNumber": "",
      "taxIdentificationNumber": "",
      "address": "George Town, Cayman Islands",
      "countryId": "58",
      "leiNumber": "",
      "urls": [
        { "urlTypeId": "7", "url": "https://exampledex.io" }
      ],
      "socials": [],
      "metadata": {
        "sources": [
          {
            "url": "https://exampledex.io/terms",
            "snippet": "ExampleDEX Labs Inc., incorporated in Cayman Islands."
          }
        ],
        "reasoning": "Entity name and jurisdiction from Terms of Service."
      }
    }
  ],
  "feedback": "ExampleDEX is a DeFi project on Ethereum with 1 DEX product, 1 governance token (EXD, ERC-20). Entity incorporated in Cayman Islands March 2023. Platform launched September 2023."
}
```

---

## Arrays That Do NOT Exist at Top Level

These were documented in older versions but are **wrong**. They nest inside parent objects:

| Old Top-Level Array | Correct Location |
|---|---|
| `urls` | Inside `profileInfos[]`, `products[]`, `assets[]`, and `entities[]` |
| `socials` | Inside `profileInfos[]`, `products[]`, `assets[]`, and `entities[]` |
| `media` | Inside `profileInfos[]`, `products[]`, and `assets[]` |
| `smartContractDeployments` | Inside `assetDeployments[]` entries (wrapped in `smartContractDeployment` object) |
| `smartContracts` | Inside `smartContractDeployment` objects |
| `assetDeployments` | Inside `assets[]` (NOT at top level) |
| `productDeployments` | Inside `products[]` |
| `productAssetRelationships` | Inside `products[]` |
| `supportsProducts` | Inside `products[]` |
| `derivativeAssets` | Structure unclear — omit unless confirmed |
| `roots` | Does not exist |
| `rootRelationships` | Does not exist |

## Fields That Do NOT Exist

| Old Field | Notes |
|---|---|
| `rootId` | Not used anywhere |
| `deploymentTypeId` | Use `deploymentType: { "id": "..." }` |
| `deployedOnId` | Use `deployedOnProduct: { "id": "..." }` |
| `deployedOnProductId` | Use `deployedOnProduct: { "id": "..." }` |
| `smartContracts[].deploymentId` | Implicit from nesting |
| `productAssetRelationships[].productId` | Implicit from nesting |
| `supportsProducts[].productId` | Implicit from nesting |
| `profileInfos[].logo` | Use `media` array with appropriate mediaTypeId |
| `profileInfos[].icon` | Use `media` array with Icon mediaTypeId |
| `assets[].address` | Use `smartContracts[].address` inside deployments |
| `assets[].smartContractDeployments` | Use `assetDeployments` with `smartContractDeployment` wrapper |
