# TGS Field Descriptions Reference

Detailed field-level documentation for the TGS schema. Organized to match
the TGS JSON output structure. Use alongside `tgs-json-structure.md` (for
nesting rules) and `tgs-enums.md` (for enum ID lookups).

---

## Profile Fields

**Profiles lens:** Top-level identities in the Grid — the organizations, projects, DAOs, investors, and collections that make up the blockchain ecosystem. Each profile carries descriptive metadata, sector and type classifications, operational status, and can be tagged for additional categorization.

### profileInfos

*Descriptive metadata for profiles — names, descriptions, classifications, and founding dates.*

| Field | Description |
|-------|-------------|
| `name` | The full official name of the profile as used in its branding and documentation. |
| `profileTypeId` | The primary operational category or organizational model of the profile. References the `profileTypes` enum table. |
| `profileSectorId` | The primary industry or problem space the profile operates in. References the `profileSectors` enum table. |
| `profileStatusId` | The current operational state of the profile. References the `profileStatuses` enum table. |
| `foundingDate` | Date when the profile was first established, registered, or publicly announced. ISO 8601 format (YYYY-MM-DD or YYYY-MM). |
| `descriptionShort` | Brief, objective overview of the profile's primary purpose or service. Hard limit 200 characters. No subjective claims, marketing language, or unnecessary jargon. |
| `descriptionLong` | Detailed overview of the profile covering its mission, target audience, and key features or offerings. Hard limit 500 characters. Objective and factual — no subjective claims. |
| `descriptionMarketing` | Promotional description targeting potential users or customers. Persuasive language and value-driven messaging are permitted here, unlike other description fields. Should highlight unique selling points and market advantages while remaining factually accurate. |
| `tagLine` | Brief, memorable phrase or slogan conveying the profile's mission or value proposition. Normalize formatting to single line, single spaced, sentence case. |

### Profile Enum Tables

**profileSectors** — *Industry and domain classifications for profiles.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this sector covers, including its scope, typical activities, and how it differs from adjacent sectors. |
| `name` | Display name of the sector. |
| `slug` | URL-safe, lowercase identifier derived from the name. Used for filtering in GraphQL queries. Must not contain spaces. |

**profileStatuses** — *Operational lifecycle stages for profiles.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this operational state means and the criteria for a profile to be classified at this stage. |
| `name` | Display name of the profile lifecycle stage. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

**profileTypes** — *Organizational model classifications for profiles.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this organizational model entails and how profiles of this type typically operate. |
| `name` | Display name of the profile type. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

### Tags

**profileTags** — *Join table linking profiles to tags for additional categorization beyond sector and type.*

| Field | Description |
|-------|-------------|
| `tagId` | References the tag applied to this profile. Points to a record in `tags`. |

**tags** — *Individual tags for flexible categorization beyond the fixed type/sector/status fields.*

| Field | Description |
|-------|-------------|
| `name` | Display name of the tag. |
| `description` | Short explanation of what this specific tag represents and when it should be applied. |
| `tagTypeId` | The category this tag belongs to. References the `tagTypes` enum table. |
| `isArchived` | Whether this tag has been archived and is no longer actively used. String boolean: "1" = archived, "2" = active. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

**tagTypes** — *Classification categories for tags.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this tag category covers and what kind of information tags of this type convey. |
| `name` | Display name of the tag category. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

---

## Product Fields

**Products lens:** Specific offerings, services, and platforms operated by blockchain profiles — exchanges, wallets, bridges, oracles, L1/L2 networks, lending protocols, and more. Each product record captures its name, type classification, lifecycle status, launch date, and relationships to other products it supports.

### products

*Individual products, services, or platforms operated under a profile.*

| Field | Description |
|-------|-------------|
| `name` | The official name of the product as used in its branding and user-facing materials. |
| `productTypeId` | The primary functional classification of the product. References the `productTypes` enum table. |
| `productStatusId` | The current lifecycle stage of the product. References the `productStatuses` enum table. |
| `description` | Concise, objective statement explaining what the product does and who it is for. Two to three sentences following Identity → Mechanism → Capability. Hard limit 500 characters, quality target 300–450. Must end with a period. |
| `isMainProduct` | Whether this is the profile's primary or flagship product. String boolean: "1" = true, "2" = false. Only one product per profile should be set to "1". |
| `launchDate` | Date when the product was first launched or made publicly available. ISO 8601 format (YYYY-MM-DD or YYYY-MM). |
| `supportsProducts` | Directional support relationships to other products. Indicates which products this product explicitly integrates with or operates on. The direction matters — a wallet supports a chain, not the reverse. Not used for on-chain deployment relationships. |

### supportsProducts

*Directional support relationships between products. The relationship has direction — the first product supports the second, not the reverse.*

| Field | Description |
|-------|-------------|
| `productId` | The product providing the support or integration. References a record in `products`. |
| `supportsProductId` | The product being supported or integrated with. References a record in `products`. |

### productDeployments

*Links products to their on-chain smart contract deployments. Used when a product has its own protocol-level deployment separate from its associated asset deployments.*

| Field | Description |
|-------|-------------|
| `deploymentId` | References the smart contract deployment record. Points to a record in `smartContractDeployments`. |
| `productId` | References the product this deployment belongs to. Points to a record in `products`. |

### Product Enum Tables

**productStatuses** — *Lifecycle stages for products, from announcement through active operation to deprecation.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this lifecycle stage means and the criteria a product must meet to be classified at this stage. |
| `name` | Display name of the product lifecycle stage. |
| `slug` | URL-safe, lowercase identifier. Used for filtering in GraphQL queries. Must not contain spaces. |

**productTypes** — *Classification categories for products based on their primary function within the blockchain ecosystem.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this product category represents, including its core functionality, typical characteristics, and how it differs from adjacent types. |
| `name` | Display name of the product category. |
| `slug` | URL-safe, lowercase identifier. Used for filtering in GraphQL queries. Must not contain spaces. |

---

## Asset Fields

**Assets lens:** Individual digital assets — tokens, coins, NFTs, and other on-chain instruments — tracked across the blockchain ecosystem. Each asset record captures its official name, trading ticker, classification type, lifecycle status, and a concise description, and links to its on-chain deployments across chains.

### assets

*Individual digital assets — tokens, coins, NFTs, and other on-chain instruments.*

| Field | Description |
|-------|-------------|
| `name` | The official name of the asset as used in its branding, marketing, and trading materials. |
| `ticker` | The unique trading symbol used to represent the asset on exchanges and trading platforms. Capital letters and numbers only. |
| `description` | A concise, objective summary of the asset's purpose, key characteristics, and use case. Target 3 sentences, though 2 is acceptable if the asset is simple. Starts with the ticker in ALL CAPS. Target 280–350 characters, hard limit 500. Avoids subjective claims and marketing language. |
| `assetTypeId` | The primary classification of the asset based on its function and economic role. References the `assetTypes` enum table. |
| `assetStatusId` | The current lifecycle stage of the asset. References the `assetStatuses` enum table. |

### derivativeAssets

*Tracks relationships between base assets and their derivative forms — wrapped tokens, liquid staking tokens, bridged versions, and synthetic representations.*

| Field | Description |
|-------|-------------|
| `baseAssetId` | The original underlying asset from which the derivative is created. References a record in the `assets` table. |
| `derivativeAssetId` | The derived asset that represents or is backed by the base asset. References a record in the `assets` table. |

### Asset Enum Tables

**assetStatuses** — *Lifecycle stages for assets, from pre-launch through active use to discontinuation.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this lifecycle stage means and the criteria an asset must meet to be classified at this stage. |
| `name` | Display name of the asset lifecycle stage. |
| `slug` | URL-safe, lowercase identifier. Used for filtering in GraphQL queries. Must not contain spaces. |

**assetTypes** — *Classification categories for digital assets based on their primary function and economic role.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this asset category represents, including its primary function, typical characteristics, and how it differs from other asset types. |
| `name` | Display name of the asset category. |
| `slug` | URL-safe, lowercase identifier. Used for filtering in GraphQL queries. Must not contain spaces. |

---

## Entity Fields

**Entities lens:** Legal structures and registered organizations behind blockchain profiles. Captures corporate identity, jurisdiction, registration details, and tax identifiers for the companies, foundations, DAOs, and other legal bodies operating in the ecosystem.

### entities

*Legal bodies behind blockchain profiles — companies, foundations, associations, and other incorporated structures.*

| Field | Description |
|-------|-------------|
| `name` | The legal name of the entity as registered with regulatory bodies or government agencies. |
| `countryId` | The country of legal registration or incorporation. References the `countries` table using ISO 3166-1 alpha-2 codes. |
| `entityTypeId` | The legal structure or registration category of the entity. References the `entityTypes` enum table. |
| `dateOfIncorporation` | Date of legal registration or incorporation. ISO 8601 format (YYYY-MM-DD). |
| `address` | The registered physical address of the entity, including street, city, state/province, and postal code where applicable. |
| `leiNumber` | The Legal Entity Identifier (LEI) assigned by GLEIF, if available. 20-character alphanumeric code. |
| `localRegistrationNumber` | The registration number assigned by the local regulatory body or government agency in the entity's jurisdiction, if available. |
| `taxIdentificationNumber` | Tax identification number assigned by the relevant tax authority, if available. |
| `tradeName` | Additional trading names, brands, or DBAs associated with the entity, separate from its legal name. |

### Entity Enum Tables

**entityTypes** — *Legal structure classifications for entities.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this legal structure entails, including its governance characteristics and typical use cases. |
| `name` | Display name of the legal structure classification. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

**countries** — *ISO 3166-1 country reference table used for entity jurisdiction lookups.*

| Field | Description |
|-------|-------------|
| `code` | ISO 3166-1 alpha-2 country code (two uppercase letters). |
| `name` | Official name of the country. |

---

## URLs

**URLs lens:** Web links associated with profiles, products, assets, and entities — homepages, documentation, app URLs, token pages, block explorer links, and more. Each URL is classified by type and constrained by which types are valid for which parent table.

### urls

*Individual URL records linked to profiles, products, assets, or entities via polymorphic references. Each URL is typed to indicate what it points to.*

| Field | Description |
|-------|-------------|
| `url` | The full URL. Must include protocol prefix (http:// or https://). |
| `urlTypeId` | The classification of this URL's content. Must be valid for the parent table per `allowedUrlTypes` constraints. References the `urlTypes` enum table. |
| `rowId` | The ID of the specific record this URL belongs to. Combined with `tableId`, resolves the parent entity. |
| `tableId` | Identifies which data table the parent entity lives in. Combined with `rowId`, resolves the parent entity. |

**urlTypes** — *Classification categories for URLs based on the content they link to.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this URL category covers, including what kind of content the link should point to and in which contexts it is valid. |
| `name` | Display name of the URL classification. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

**allowedUrlTypes** — *Constraint rules defining which URL types are valid for which parent tables.*

| Field | Description |
|-------|-------------|
| `tableId` | Identifies which data table this rule applies to. Determines which parent entity type can carry this URL type. |
| `urlTypeId` | The URL type being permitted for this table. References a record in `urlTypes`. |

---

## Socials

**Socials lens:** Social media presences linked to profiles, products, and assets. Tracks platform accounts (Twitter, Discord, GitHub, Telegram, etc.) with their handles, operational status, and parent entity references.

### socials

*Individual social media account records linked to profiles, products, or assets via polymorphic references.*

| Field | Description |
|-------|-------------|
| `name` | The handle or username for this social media account. Should match the handle displayed on the platform profile page (typically the last segment of the profile URL). |
| `socialTypeId` | The social media platform this account is on. References the `socialTypes` enum table. |
| `socialStatusId` | The current operational state of this social account. References the `socialStatuses` enum table. |
| `rowId` | The ID of the specific record this social account belongs to. Combined with `tableId`, resolves the parent entity. |
| `tableId` | Identifies which data table the parent entity lives in. Combined with `rowId`, resolves the parent entity. |

**socialTypes** — *Social media platform classifications.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this social media platform is and how it is typically used within the blockchain ecosystem. |
| `name` | Display name of the social media platform. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

**socialStatuses** — *Operational states for social media accounts.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this operational state means for a social media account. |
| `name` | Display name of the social account status. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

---

## Media

**Media lens:** Visual assets associated with profiles, products, and assets — logos, icons, headers, and screenshots. Each media record links an image URL to a specific entity via polymorphic table/row references.

### media

*Image records linked to profiles, products, or assets via polymorphic references.*

| Field | Description |
|-------|-------------|
| `url` | Direct URL to the image file. |
| `mediaTypeId` | The classification of this image. References the `mediaTypes` enum table. |
| `rowId` | The ID of the specific record this image belongs to. Combined with `tableId`, resolves the parent entity. |
| `tableId` | Identifies which data table the parent entity lives in. Combined with `rowId`, resolves the parent entity. |

**mediaTypes** — *Classification categories for visual assets.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this media category represents and where it is typically displayed. |
| `name` | Display name of the media classification. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

---

## Smart Contract Deployments

**Smart Contract Deployments lens:** On-chain deployment records linking products and assets to their smart contracts across blockchains. Tracks which chain a contract is deployed on, what token standard it implements, its deployment purpose, and the individual contract addresses.

### smartContractDeployments

*Deployment records that group related smart contracts under a single chain, standard, and purpose. Links upward to products (via `productDeployments`) and assets (via `assetDeployments`).*

| Field | Description |
|-------|-------------|
| `deployedOnId` | The blockchain network this deployment lives on. References a product record representing the chain. Blank if there is no on-chain deployment. |
| `deploymentTypeId` | The primary purpose of this deployment. References the `deploymentTypes` enum table. |
| `assetStandardId` | The token or contract standard this deployment implements. References the `assetStandards` enum table. Blank if the standard cannot be identified. |

### smartContracts

*Individual smart contract addresses belonging to a deployment. A single deployment can contain multiple related contracts.*

| Field | Description |
|-------|-------------|
| `address` | The on-chain address of this smart contract. Format must match the parent blockchain's address structure. |
| `name` | The identifier or label for this contract as used in project documentation or block explorers. Blank if unnamed. |
| `deploymentDate` | Date when this contract was deployed on-chain. ISO 8601 format (YYYY-MM-DD). |
| `deploymentId` | Links this contract to its parent deployment record. References a record in `smartContractDeployments`. A deployment can contain multiple related contracts, but each chain requires its own deployment record. |

### Deployment Enum Tables

**deploymentTypes** — *Classification of a deployment's primary purpose.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this deployment purpose entails and when it should be applied. |
| `name` | Display name of the deployment type. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

**assetStandards** — *Token and contract standard specifications that smart contracts can implement.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this token standard defines, including its interface, capabilities, and typical use cases. |
| `name` | Display name of the token or contract standard. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

### assetDeployments

*Links assets to their on-chain smart contract deployments across different blockchains.*

| Field | Description |
|-------|-------------|
| `assetId` | References the asset this deployment belongs to. Points to a record in the `assets` table. |
| `deploymentId` | References the smart contract deployment record. Points to a record in the `smartContractDeployments` table. |

---

## Product-Asset Relationships

**Product Asset Relationships lens:** Links between products and the assets they interact with, classified by the nature of the relationship — native tokens, supported assets, governance tokens, payment methods, and more.

### productAssetRelationships

*Individual links between a product and an asset, each classified by how the asset is used within that product.*

| Field | Description |
|-------|-------------|
| `assetId` | The asset involved in this relationship. References a record in `assets`. |
| `productId` | The product involved in this relationship. References a record in `products`. |
| `assetSupportTypeId` | The nature of the relationship between the product and asset. References the `assetSupportTypes` enum table. |

**assetSupportTypes** — *Classification of how an asset relates to a product.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this relationship type means and how the asset functions within the product under this classification. |
| `name` | Display name of the support relationship type. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

---

## Internal / System Tables

**Internal Helpers lens:** System-level reference tables and infrastructure that support the Grid's data model. Includes schema metadata, ranking scores, inter-profile relationships, and root-level profile records.

### roots

*Top-level profile records — the canonical identity anchor for every profile in the Grid. All other data hangs off a root.*

| Field | Description |
|-------|-------------|
| `slug` | URL-friendly identifier auto-generated from the profile name. Lowercase letters, numbers, and hyphens only. |
| `urlMain` | The primary website URL for this profile. |
| `firstPublicValidation` | Date when this profile was first publicly validated. ISO 8601 format. |
| `lastPublicValidation` | Date of the most recent public validation for this profile. ISO 8601 format. |

### rootRelationships

*Hierarchical links between profiles, typically representing brand/sub-brand or parent/subsidiary structures.*

| Field | Description |
|-------|-------------|
| `parentRootId` | The overarching profile in the relationship, typically the parent brand or organization. References a record in `roots`. |
| `childRootId` | The subordinate profile in the relationship. References a record in `roots`. |
| `relationshipTypeId` | The nature of the relationship between parent and child. References the `rootRelationshipTypes` enum table. |

**rootRelationshipTypes** — *Classification of hierarchical relationships between profiles.*

| Field | Description |
|-------|-------------|
| `definition` | Explanation of what this relationship type represents and how the parent and child profiles are expected to relate. |
| `name` | Display name of the relationship type between parent and child profiles. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

### gridRank

*Importance scoring for profiles based on ecosystem connectivity.*

| Field | Description |
|-------|-------------|
| `score` | Numeric importance score for a profile. Higher is better. Currently derived from connection count — how many other profiles, products, and assets a profile is linked to. |

### Schema Metadata

**coreTableNames** — *Registry of primary data tables in the schema.*

| Field | Description |
|-------|-------------|
| `tableName` | Name of a primary data table in the Grid schema. |

**enumTableNames** — *Registry of enum/reference tables in the schema.*

| Field | Description |
|-------|-------------|
| `enumTableName` | Name of an enum or reference table in the Grid schema. |

---

## Attributes

**Attributes lens:** Flexible key-value metadata system for tagging profiles and entities with structured properties that don't fit into fixed schema fields. Attribute types define the categories, constraint rules govern which types are valid for which entity classes, and attribute records store the actual values.

*Rarely used during standard enrichment. Included for completeness.*

### attributes

*Actual attribute values assigned to profiles.*

| Field | Description |
|-------|-------------|
| `attributeTypeId` | References the type of attribute being recorded. Points to a record in `attributeTypes`. |
| `value` | The attribute's actual value for this entity. Format depends on the attribute type definition. |
| `rowId` | Identifies the specific record this attribute is attached to. Combined with `tableId`, pinpoints the exact entity. |
| `tableId` | Identifies which data table the target entity lives in. Combined with `rowId`, resolves the entity this attribute describes. |

**attributeTypes** — *Definitions for the categories of attributes that can be applied to entities.*

| Field | Description |
|-------|-------------|
| `definition` | Detailed explanation of what this attribute measures or certifies, including expected value formats and how it should be interpreted. |
| `name` | Display name of the attribute type. |
| `slug` | URL-safe, lowercase identifier. Must not contain spaces. |

**allowedAttributeTypes** — *Constraint rules defining which attribute types are valid for specific entity classes.*

| Field | Description |
|-------|-------------|
| `attributeTypeId` | References the attribute type being permitted. Points to a record in `attributeTypes`. |
| `tableId` | Identifies the primary data table this rule applies to. |
| `enumTableId` | Identifies the enum table this rule applies to. Scopes the constraint to a specific classification system. |
| `enumRowId` | Identifies the specific enum value within the table that this attribute type is allowed on. Combined with `enumTableId`, defines exactly which entity class can carry this attribute. |
