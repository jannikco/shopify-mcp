# Shopify MCP server

Inspect store products, then explicitly approve a title change to one chosen product.

A direct-provider MCP server with useful local execution. No PipMCP account is required for local use. Provider API access and any provider charges are your responsibility.

## Quick start

Requires Node.js22+. Clone this repository, then run:

```sh
npm ci --ignore-scripts
npm test
npm start
```

The default command inspects the tools offline. Configure your own credentials in your environment:

- `SHOPIFY_ACCESS_TOKEN`
- `SHOPIFY_STORE`



Credential help: https://shopify.dev/docs/api/admin-graphql/2026-10/queries/products

## Provider setup

Use your own installed Shopify app Admin access token. SHOPIFY_STORE is only the myshopify.com store label. Grant read_products for reads and write_products for approved title updates. Responses may contain GraphQL errors/userErrors even with HTTP200; inspect these before treating a change as successful. For managed GraphQL import, retrieve a version2026-10 schema from your own authenticated Admin GraphQL endpoint using standard __schema introspection as described in the official About GraphQL reference, or use Shopify GraphiQL explorer. This package does not bundle a vendor schema.

## Execute a real read

Replace example record IDs with records you are authorized to access.

```sh
npm start -- --call shopify_list_products '{"first":5}'
```

## Use with an MCP client

Launch `node /absolute/path/shopify-mcp/server.mjs` using stdio transport; pass credentials through the client environment. Stdout is reserved for JSON-RPC.

## Tools

| Tool | Effect | Purpose |
| --- | --- | --- |
| shopify_list_products | Read | Read a bounded product page using a fixed Admin GraphQL query. |
| shopify_update_product_title | Explicit approval required | Update only the chosen product title. Requires write_products scope; inspect returned userErrors. |

## Concrete workflow

Inspect store products, then explicitly approve a title change to one chosen product.

1. shopify_list_products
2. shopify_update_product_title

## Optional writes

Writes are disabled by default. Set CONNECTOR_ALLOW_WRITES=true and supply `_approval: {approved:true,idempotencyKey:"unique-request-id"}` on the specific call. Do not reuse a key for a different request. Uncertain outcomes are never automatically replayed; inspect the provider before another attempt.

## Capability evidence

Local: **Tested — HTTP fixtures and real stdio tools/call**. Managed: **Documented**. Provider authentication and live operations have not been established by these fixture tests. Evidence applies per operation and execution mode; setup discovery does not imply provider execution.

## Managed production

[Import with PipMCP](https://pipmcp.com/github/start?integration=graphql-to-mcp&utm_source=github&utm_campaign=github-shopify-mcp&utm_content=readme) when you want the hosted service. This provider is not currently listed as a managed template. Import selected-openapi.json (REST) or the official GraphQL schema linked in provenance, configure authentication, and review the tools before execution. Local credentials are not uploaded by this code. Review provider permissions in the managed onboarding. [Directory and related projects](https://jannikco.github.io/pipmcp-connectors/integrations/shopify/).

## Provenance and maintenance

- https://shopify.dev/docs/api/admin-graphql/2026-10/queries/products
- https://shopify.dev/docs/api/admin-graphql/2026-10/mutations/productUpdate
- https://shopify.dev/docs/apps/build/graphql

Maintainer:jannikco. MIT license for this code. See connector.json for selected operations and integration.json for evidence. Runtime dependencies retain their own licenses. Report sensitive issues through https://pipmcp.com/contact; do not post credentials or customer data in public issues.
