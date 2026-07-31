---
name: mori-shop-catalog
description: Browse the MORI (babymori.com) baby & children's catalog and build a cart on behalf of a user via the store's live Shopify Storefront MCP server. Read-only browsing and cart building require no authentication; payment must always be human-approved.
api: MORI Storefront MCP (https://babymori.com/api/mcp)
transport: streamable-http
protocol_version: '2025-06-18'
operations:
- search_catalog
- get_product_details
- update_cart
- get_cart
- search_shop_policies_and_faqs
generated: '2026-07-20'
method: generated
source: grounded in the live tools/list of https://babymori.com/api/mcp (storefront-renderer 0.1.0)
---

# Shop the MORI catalog (agent flow)

MORI's store at `https://babymori.com` exposes a hosted Shopify Storefront MCP
server at `POST https://babymori.com/api/mcp`. All tool names below are real and
verified live. No auth token is needed to browse or build a cart; completing
payment always requires explicit buyer approval.

## Steps

1. **Find products.** Call `search_catalog` with a natural-language `catalog.query`
   (e.g. "organic cotton baby sleepwear 0-3 months"). Pass buyer context
   (`context.address_country`, `context.currency`) for accurate pricing. Results
   are paginated — use `pagination.cursor` to fetch more.
2. **Inspect a product.** Call `get_product_details` with the returned
   `product_id` (a `gid://shopify/Product/...`). Pass `options`
   (e.g. `{"Size":"0-3","Color":"Grey"}`) to select a variant; omit to get the
   first available variant.
3. **Answer policy questions.** For returns, shipping, hours, or contact, call
   `search_shop_policies_and_faqs` with a natural-language `query` — do not guess.
4. **Build the cart.** Call `update_cart` with `add_items` (each a
   `product_variant_id` + `quantity`). Omit `cart_id` to create a new cart; reuse
   the returned id for subsequent edits. Set `buyer_identity`, delivery addresses,
   and delivery options as the user provides them.
5. **Review.** Call `get_cart` with the `cart_id` to confirm items, shipping
   options, discounts, and the `checkout_url`.
6. **Hand off for payment.** Present the `checkout_url` to the user. Do NOT
   complete payment autonomously — the store's agent policy (see `/llms.txt` and
   `/.well-known/ucp`) requires contemporaneous buyer approval, or routing the
   purchase through the Shop skill (`https://shop.app/SKILL.md`) via Shop Pay.

## Rules

- Respect rate limits: back off on HTTP 429.
- Only apply `discount_codes` / `gift_card_codes` the user actually mentions.
- Never fabricate product ids, variants, or prices — always read them from
  `search_catalog` / `get_product_details`.
