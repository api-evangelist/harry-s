# Mammoth Brands

Mammoth Brands is the consumer packaged goods company formerly known as Harry's Inc., renamed in
April 2025 to reflect a portfolio spanning Harry's, Flamingo, Lume, Mando and Coterie.

There is no developer programme, no portal and no OpenAPI. What there is — and it is real — is an
agentic-commerce surface on the two brands that run on Shopify:

- **Harry's** — anonymous MCP server at `https://www.harrys.com/api/mcp` (5 tools, protocol
  `2025-06-18`) and a UCP merchant profile at `/.well-known/ucp.json` (version `2026-04-08`).
- **Flamingo** — the same MCP surface at `https://www.shopflamingo.com/api/mcp`, plus a published
  `llms.txt` and `agents.md`.
- **Harry's + Flamingo** — Shopify Customer Account API OIDC / RFC 8414 discovery documents served
  from their own apex domains.
- **Lume, Mando, Coterie** — no agent surface.

No A2A agent card, `security.txt`, status page, changelog, SDK, CLI or Postman collection exists on
any Mammoth Brands host as of 2026-07-31.

- https://www.mammothbrands.com/
