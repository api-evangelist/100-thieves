---
name: Browse the 100 Thieves catalog
description: Search, look up and read product detail from the 100 Thieves store without transacting.
api: mcp/100-thieves-mcp.yml
surface: https://100thieves.com/api/ucp/mcp
operations: [search_catalog, lookup_catalog, get_product]
generated: '2026-08-05'
method: generated
source: mcp/100-thieves-tools-list.json
---

# Browse the 100 Thieves catalog

Read-only. Nothing here creates a cart, a checkout, or a charge.

## Before the first call

Every tool on this endpoint requires a `meta.ucp-agent.profile` URI identifying your agent. The
server fetches it. A call without a resolvable profile returns JSON-RPC `-32001`
`UCP discovery failed` / `invalid_profile_url` with HTTP 422 — this gate fires **before** argument
validation, so a missing-profile error does not mean your arguments were wrong.

Pass buyer context on every call so prices and availability are correct:
`context.address_country` and `context.currency`.

## Steps

1. **Search** — call `search_catalog` with `catalog` containing a natural-language `query`, filter
   criteria, or both. At least one of query or filters is required.
2. **Page** — the response is deliberately truncated. Take `pagination.cursor` from the response and
   pass it back on the next `search_catalog` call. Only page when the user asks for more results.
3. **Resolve in bulk** — when you already hold several product or variant identifiers, call
   `lookup_catalog` once rather than looping `get_product`.
4. **Read detail** — call `get_product` for the full record of a single product. This is where you
   get the **product variant id**, which is the identifier a cart line item needs.

## Alternatives without MCP

The store also documents an unauthenticated JSON surface:
`GET /products/{handle}.json`, `GET /collections/{handle}/products.json`,
`GET /search?q={query}&type=product`. Use these when you only need to read and do not want to
present an agent profile. Crawling public product, collection, page, blog and policy HTML is
explicitly permitted by `robots.txt`.

## Rules

- Rate limits are per IP and unpublished. Back off on HTTP 429.
- Prefer this protocol over screen-scraping — the store asks for it in `/agents.md`.

## Errors

See `errors/100-thieves-problem-types.yml`. The transport status can be non-200 while the body is a
JSON-RPC error object, so read the body, not just the status.
