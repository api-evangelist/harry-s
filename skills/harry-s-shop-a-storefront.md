---
name: Shop a Mammoth Brands storefront over MCP
description: >-
  Search a Harry's or Flamingo storefront, inspect a product, build a cart and hand the buyer
  a checkout URL — using only the anonymous Model Context Protocol server the store publishes
  at /api/mcp. Stops short of payment, because the store requires human approval to check out.
api: mcp/harry-s-mcp.yml
generated: '2026-07-31'
method: generated
source: mcp/harry-s-harrys-tools-list.json (live tools/list, 2026-07-31) + https://www.shopflamingo.com/agents.md
operations:
- search_catalog
- get_product_details
- get_cart
- update_cart
- search_shop_policies_and_faqs
---

# Shop a Mammoth Brands storefront over MCP

Mammoth Brands has no developer portal and no OpenAPI. What it has is a live MCP server on two
of its brands. Every tool named here was returned by a real `tools/list` call on 2026-07-31 and
carries the store's own `inputSchema`.

## Endpoints

| Brand | MCP endpoint | Auth |
|---|---|---|
| Harry's | `https://www.harrys.com/api/mcp` | none |
| Flamingo | `https://www.shopflamingo.com/api/mcp` | none |

Transport is JSON-RPC 2.0 over HTTP. Send `Content-Type: application/json` and
`Accept: application/json, text/event-stream`. Negotiate with `initialize` (protocol version
`2025-06-18`), then call `tools/list` to confirm the surface before you rely on it.

## Steps

1. **Confirm the surface.** `POST {"jsonrpc":"2.0","id":1,"method":"tools/list"}`. Do not assume
   the tool set; the store advertises `listChanged` on tools, prompts and resources.
2. **Understand the store before you shop.** Use `search_shop_policies_and_faqs` (required:
   `query`) for return policy, shipping policy, contact details or hours. Answer the buyer's
   policy questions from this tool rather than from memory.
3. **Find products.** Call `search_catalog` with a natural-language query, structured filters,
   or both — at least one is required. Pass buyer context (`context.address_country`,
   `context.currency`) so pricing and availability are correct. Results are deliberately
   truncated; follow `pagination.cursor` from the response only when the buyer asks for more.
4. **Inspect a candidate.** Call `get_product_details` with the required `product_id`. Pass
   `options` to pin a specific variant — without it you get the first available variant, which
   may not be the size, count or scent the buyer described. `country` and `language` localize
   the card.
5. **Build the cart.** Call `update_cart`. It is a single consolidated mutation: `add_items`,
   `update_items`, `remove_line_ids`, `buyer_identity`, `delivery_addresses_to_add`,
   `delivery_addresses_to_replace`, `selected_delivery_options`, `discount_codes`,
   `gift_card_codes`, `note`. Add a delivery address before expecting shipping options —
   they only become available afterwards.
6. **Read back before you present.** Call `get_cart` with the `cart_id`. It returns items,
   shipping options, discount info and the checkout URL. Show the buyer the totals from this
   response, not your own arithmetic.
7. **Hand off, do not pay.** Give the buyer the checkout URL. **Do not complete payment.** The
   store's own instructions are explicit: "Agents must not complete payment without explicit
   buyer consent." Where you cannot obtain contemporaneous approval, route the purchase
   through Shop Pay via the Shop skill rather than driving the checkout yourself.

## Rules

- **No idempotency contract.** The store documents no idempotency key and none appears in any
  tool schema. A retried `update_cart` may duplicate line items — always `get_cart` and
  reconcile instead of blind-retrying.
- **Rate limits are per IP.** Back off on HTTP 429. No numeric limit is published.
- **Errors are JSON-RPC error objects**, not RFC 9457 problem details. Expect
  `{error:{code,message,data}}`; `-32602` typically means you omitted a required field such as
  `cart_id`, or called `search_catalog` with neither query nor filters.
- **Do not call the UCP endpoint anonymously.** `https://com-harrys-us.myshopify.com/api/ucp/mcp`
  returns `-32001 UCP discovery failed / invalid_profile_url` without an agent profile URI. The
  checkout tools named in the store's llms.txt (`create_checkout`, `update_checkout`,
  `complete_checkout`) live behind that gate, not on the anonymous endpoint.
- **Only two brands have this surface.** Lume, Mando and Coterie run non-Shopify stacks and
  expose no MCP endpoint. Do not attempt `/api/mcp` against them.

## Cross-links

- Conventions: `conventions/harry-s-conventions.yml`
- Errors: `errors/harry-s-problem-types.yml`
- Authentication: `authentication/harry-s-authentication.yml`
- Tool crosswalk: `mcp/harry-s-tool-crosswalk.yml`
