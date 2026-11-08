# Shopify product workflows for AI assistants

Bring your product catalog into your AI workspace and turn a reviewed merchandising decision into a targeted title update.

[Start locally](#get-started) · [Connect your assistant](#connect-your-assistant) · [Available tools](#available-tools) · [Hosted PipMCP setup](#use-pipmcp-for-production)

## Built for useful workflows

- **Prepare a product brief.** Read a bounded page of product titles and status before discussing merchandising changes.
- **Review a title refresh.** Use the product IDs returned by Shopify to choose a title change, then approve that exact update.
- **Build a repeatable catalog review.** Use the same product tools from a CLI or MCP assistant, with separate read and write permissions.

## Why use this package?

Get a focused Shopify MCP integration without assembling request mappings and a stdio transport yourself. The package provides documented input schemas, provider authentication handling and an example CLI, with the same tools available to your assistant.

Reads are available independently of writes. Approved changes use operator opt-in and a per-call idempotency key; uncertain writes are not silently replayed.

## Get started

**Requirements:** Node.js 22 or newer, npm, and access to the provider account you intend to use.

```sh
git clone https://github.com/jannikco/shopify-mcp.git
cd shopify-mcp
npm ci --ignore-scripts
npm test
npm start
```

`npm start` shows the tool inventory and workflow offline. It does not authenticate or call the provider. The tests use provider fixtures and stdio transport; they do not modify your account.

### Configure your provider account

The current release connects directly to the provider with **your own credentials**. It does not require a PipMCP key for local execution. Configure the environment of the process or MCP client that runs the server; `.env` files are not loaded automatically.

| Environment variable | What to supply |
| --- | --- |
| `SHOPIFY_ACCESS_TOKEN` | Your own provider API key or access token. |
| `SHOPIFY_STORE` | Your authorized account/store/region label for the API host; see provider setup. |

Use your own installed Shopify app Admin access token. SHOPIFY_STORE is only the myshopify.com store label. Grant read_products for reads and write_products for approved title updates. Responses may contain GraphQL errors/userErrors even with HTTP 200; inspect these before treating a change as successful. For managed GraphQL import, retrieve a version 2026-10 schema from your own authenticated Admin GraphQL endpoint using standard __schema introspection as described in the official About GraphQL reference, or use Shopify GraphiQL explorer. This package does not bundle a vendor schema.

[Provider credential/reference documentation](https://shopify.dev/docs/api/admin-graphql/2026-10/queries/products). Keep credentials out of commits, issue reports and shared client configuration.

### Run your first provider read

After configuring the environment, run `shopify_list_products`:

```sh
npm start -- --call shopify_list_products '{"first":5}'
```

Replace sample account, record, file and project IDs with values from your authorized account. This command makes a real provider request and returns its result for you to inspect.

## Available tools

| Capability | MCP tool | Effect | Required inputs |
| --- | --- | --- | --- |
| Read a bounded product page using a fixed Admin GraphQL query. | `shopify_list_products` | Read | `first` |
| Update only the chosen product title. Requires write_products scope; inspect returned userErrors. | `shopify_update_product_title` | Approved write | `id`, `title` |

Tool schemas and bindings are in [connector.json](./connector.json). These are the selected operations this package ships.

### A practical tool sequence

1. Use `shopify_list_products` — Read a bounded product page using a fixed Admin GraphQL query.
2. Use `shopify_update_product_title` — Update only the chosen product title. Requires write_products scope; inspect returned userErrors.

## Connect your assistant

Add this stdio server to an MCP-compatible client. Replace the absolute path and credential placeholders. Use your client’s secret/environment controls instead of committing this configuration.

```json
{
  "mcpServers": {
    "shopify-mcp": {
      "command": "node",
      "args": [
        "/absolute/path/shopify-mcp/server.mjs"
      ],
      "env": {
        "SHOPIFY_ACCESS_TOKEN": "YOUR_SHOPIFY_ACCESS_TOKEN",
        "SHOPIFY_STORE": "YOUR_SHOPIFY_STORE"
      }
    }
  }
}
```

`node server.mjs` starts the MCP server; `npm start` runs the example CLI. The server reserves stdout for MCP messages. If the client does not inherit your shell environment, pass the required variables explicitly.

## Review changes before running writes

The write tool is disabled by default. Enable `CONNECTOR_ALLOW_WRITES=true` in the server process and include this approval object in the **specific** write call after reviewing its arguments:

```json
{
  "_approval": {
    "approved": true,
    "idempotencyKey": "unique-reviewed-request-id"
  }
}
```

This object is added alongside the tool’s own required fields; it is not a complete write request. Use a new idempotency key for a different action. If an outcome is uncertain, inspect the provider before trying again. Creating a record may trigger existing provider automations even when this package does not send a message itself.

## Use PipMCP for production

The local package is useful when you want to run these selected Shopify operations in your own process. PipMCP provides a hosted place to configure provider connections, publish tools, issue endpoint access tokens, apply server-side policy and approvals, and inspect execution/usage records. Provider credentials and hosted execution are configured in that service.

**[Import Shopify into PipMCP](https://pipmcp.com/github/start?integration=graphql-to-mcp&utm_source=github&utm_campaign=github-shopify-mcp&utm_content=readme)**

This provider is not listed as a ready managed template in the current availability snapshot (2026-11-08). The link opens the GraphQL importer; it does not activate the local tools automatically.

Use the provider’s official GraphQL schema or obtain an authorized schema as described in the provider references below. Review the imported tools and configure the matching provider connection in PipMCP.

Local credentials are not uploaded by this repository. A hosted endpoint uses its own PipMCP endpoint access token and published tool inventory; the local and hosted credentials are separate.

## Scope and validation

- The installed package has HTTP request fixtures and MCP stdio `tools/call` tests.
- Fixture success checks client behavior; it does not establish provider authentication, paid-plan access or live account execution.
- No live provider execution is claimed for this package by these tests.
- Hosted tool names, schemas, permissions and availability can differ from this local package. A queued or accepted job/message is not proof of completion or delivery. Provider API plans, account rules and limits apply.
- See [integration.json](./integration.json) for operation-scoped evidence and the provider-specific limits above.

## FAQ

**Do I need a Shopify account?**

You need the provider account, token and permissions required by these operations. Provider charges and API limits remain with that account.

**Does this version require a PipMCP API key?**

This package uses your provider credentials for local execution. To use the hosted service, configure the provider connection in PipMCP and use the endpoint access token issued there.

**Why is a tool missing or a request rejected?**

This repository exposes the selected tools above. Check the required environment variables, provider scopes, account/region requirements and input IDs first. Hosted endpoints have their own published inventory and may exclude tools by role or policy. Do not retry a failed write without checking its outcome.

**Is this an official provider SDK or a hosted-service guarantee?**

This is an independently maintained MCP integration. Its fixtures and source references do not imply provider endorsement, unrestricted API access, or identical hosted behavior.

## Guides and related workflows

- [Shopify integration guide](https://jannikco.github.io/pipmcp-connectors/integrations/shopify/)
- [Shopify workflow and tool sequence](https://jannikco.github.io/pipmcp-connectors/workflows/shopify/)
- [More integration workflows](https://jannikco.github.io/pipmcp-connectors/workflows/)
- [Typed LeadAdapter API client](https://github.com/jannikco/linkedin-typescript-sdk)
- [LeadAdapter Python API client](https://github.com/jannikco/linkedin-python-sdk)
- [LinkedIn API quickstart](https://github.com/jannikco/linkedin-api)
- [LinkedIn people and post search](https://github.com/jannikco/linkedin-search-api)
- [LinkedIn post engagers](https://github.com/jannikco/linkedin-post-engagers)
- [LeadAdapter LinkedIn MCP server](https://github.com/jannikco/linkedin-mcp)
- [OpenAPI to MCP import starter](https://github.com/jannikco/openapi-to-mcp)
- [Swagger to MCP import starter](https://github.com/jannikco/swagger-to-mcp)
- [Shopify API and credential reference](https://shopify.dev/docs/api/admin-graphql/2026-10/queries/products)
- [Additional official source 2](https://shopify.dev/docs/api/admin-graphql/2026-10/mutations/productUpdate)
- [Additional official source 3](https://shopify.dev/docs/apps/build/graphql)

Maintained by [jannikco](https://github.com/jannikco). MIT for this repository’s code; dependencies retain their licenses. [Report a reproducible issue](https://github.com/jannikco/shopify-mcp/issues) with redacted input and error details. For credentials or security issues, use [private contact](https://pipmcp.com/contact).
