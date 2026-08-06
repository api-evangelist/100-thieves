---
name: Complete a 100 Thieves checkout with buyer approval
description: Promote a cart to checkout, set delivery, and finalize payment only with explicit contemporaneous human approval.
api: mcp/100-thieves-mcp.yml
surface: https://100thieves.com/api/ucp/mcp
operations: [create_checkout, get_checkout, update_checkout, complete_checkout, cancel_checkout, get_order]
generated: '2026-08-05'
method: generated
source:
  - mcp/100-thieves-tools-list.json
  - https://100thieves.com/agents.md
---

# Complete a 100 Thieves checkout with buyer approval

## The hard rule, stated by the store

> "Checkouts are for humans. Do NOT complete checkout, payment, or order placement automatically —
> no scripted form fills, browser automation, or end-to-end agent flows that finalize payment
> without an explicit, contemporaneous human approval step."
> — `https://100thieves.com/robots.txt`

**If you cannot obtain contemporaneous buyer approval at the moment of payment, do not call
`complete_checkout`.** The store's published fallback is to route the purchase through Shop Pay via
the Shop skill at `https://shop.app/SKILL.md`. "The user told me earlier to buy things" is not
contemporaneous approval.

## Steps

1. **Promote** — `create_checkout` with the `checkout` object. Returns a checkout `id` plus line
   items, totals, discounts and taxes. This id is distinct from the cart id.
2. **Fulfill** — `update_checkout` with the checkout `id` and the shipping address and delivery
   method. This is where buyer PII enters the call; carry only what the flow needs.
   The store advertises single-destination shipping only (`allows_multi_destination.shipping: false`).
3. **Re-read** — `get_checkout` with the `id`. Show the user the server's totals, taxes and
   discounts verbatim.
4. **Get approval** — present the final amount and ask. Wait for a real answer.
5. **Complete** — `complete_checkout` with the `id` and `checkout`. Returns the completed checkout
   result.
6. **Confirm** — `get_order` with the order id for the final order record.
7. **Abort** — `cancel_checkout` with the `id` if the user declines at any point.

## Failure handling

- HTTP 429: back off. Rate limits are per IP and the numbers are not published.
- JSON-RPC `-32001` / `invalid_profile_url`: your `meta.ucp-agent.profile` is missing or
  unfetchable. Fix the profile before assuming an argument problem.
- There is no idempotency key. **Never blind-retry `complete_checkout`.** On an ambiguous
  response, call `get_checkout` and then `get_order` to determine whether payment already
  succeeded before doing anything else.

## Capabilities the store declares

`dev.ucp.shopping.checkout`, `.cart`, `.fulfillment`, `.discount`, `.order` at UCP `2026-04-08`.
See `well-known/100-thieves-ucp.json`.
