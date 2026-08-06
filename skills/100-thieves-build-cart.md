---
name: Build and maintain a 100 Thieves cart
description: Create a cart, add and adjust line items, attach buyer details, and read cart totals.
api: mcp/100-thieves-mcp.yml
surface: https://100thieves.com/api/ucp/mcp
operations: [create_cart, get_cart, update_cart, cancel_cart, get_product]
generated: '2026-08-05'
method: generated
source: mcp/100-thieves-tools-list.json
---

# Build and maintain a 100 Thieves cart

This skill stops short of checkout. It creates and mutates a cart only — no payment.

## Preconditions

- `meta.ucp-agent.profile` on every call (see `100-thieves-browse-catalog.md`).
- A **product variant id**, not a product id. `cart.line_items[].item.id` is documented as
  "The Product Variant ID to add or update." Resolve it with `get_product` first.

## Steps

1. **Create** — call `create_cart` with `cart.line_items[]`, each entry carrying
   `item.id` (variant id) and `quantity`. Both are required. Keep the returned cart `id`; every
   later call needs it.
2. **Attach the buyer** — set `cart.buyer.email` and optionally `cart.buyer.phone_number`.
3. **Set context** — `cart.context` takes `address_country`, `address_region`, `postal_code`,
   `language` (BCP 47) and `currency` (ISO 4217). These are *provisional hints*: authoritative data
   such as a real shipping address supersedes them, and unsupported hints are ignored without
   error, so never treat a silently dropped hint as a failure.
4. **Adjust** — call `update_cart` with the cart `id` and the changed fields. One `update_cart`
   covers work that the Storefront GraphQL API splits across seven separate mutations, so batch
   your changes rather than issuing one call per field.
5. **Read back** — `get_cart` with the `id` returns current contents and totals. Do this before
   showing the user a number; do not compute totals yourself.
6. **Abandon** — `cancel_cart` with the `id` when the user walks away.

## No idempotency key

There is no idempotency header or token on this endpoint. Safety comes from the id contract: create
once, then address that id. **Do not retry `create_cart` on an ambiguous failure** — you will create
a second cart. Retry `get_cart` instead and reconcile.

## Next

Hand off to `100-thieves-checkout-with-buyer-approval.md`.
