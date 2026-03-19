# The Grid — Public Resources

Public skills, tools, and reference docs for [The Grid](https://thegrid.id)'s Web3 data platform.

## Contents

- **grid-search/** — Claude Code plugin for querying The Grid's database via MCP. Includes search skills and a full GraphQL query reference.
- **references/** — TGS schema docs: field descriptions, product type definitions, JSON structure, and classification guides.

## How to update

This repo is a read-only mirror of the `public/` folder in the private [workspace](https://github.com/The-Grid-Data/workspace) repo.

1. Make changes in the **workspace** repo under `public/`
2. Commit and push to workspace as normal
3. Run `./scripts/sync-public.sh` from workspace root to publish here

Do not edit this repo directly — changes will be overwritten on next sync.
