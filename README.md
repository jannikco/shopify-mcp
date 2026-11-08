# Shopify MCP server

## Review Shopify draft products

Prepare a catalog review from products still in draft, without changing titles or publishing products.

### Try the task with synthetic data

No product key is needed for this demo. Requires Node.js 22+.

```sh
npm ci --ignore-scripts
npm run demo
# Optional: save a private report to a new file.
npm run demo -- --output ./synthetic-report.json
```

Output: a task-specific JSON review report selecting 1 of 2 invented records. It includes the completed fixture receipt and a request trace; no provider request or write occurs.

The synthetic records use normalized review fields. Review and map your actual provider schema before applying the same selection to live data.

### Use your authorized account

[Activate this task](https://pipmcp.com/github/start?integration=shopify-mcp&app=shopify&flow=shopify-draft-catalog-review&utm_source=github&utm_campaign=github-shopify-mcp&utm_content=task-demo), then set **PIPMCP_API_KEY** in your process environment. Provider credentials stay in PipMCP. Configure the provider connection and explicitly select integration/connection IDs when your account has several.

```sh
npm run preflight
```

Preflight checks access/discovery only; it does not count as a completed provider task. For an explicitly reviewed read, use the returned live schema and the exact operation/tool, arguments and stable idempotency key:

```sh
npm run verify:read -- --tool '<actual discovered read-only tool>' --arguments '<reviewed argument JSON>' --idempotency-key '<stable request key>' --confirm-read-only
```

Discovery/calls require connectors:read and connectors:call on the PipMCP key. The verification command performs only the selected authorized read; it refuses writes and never selects a customer operation automatically. Only a completed receipt establishes a live read. Missing access, pending jobs and uncertain outcomes remain explicit blockers. Reports contain receipt digests rather than customer payloads. No prices, trials or account allowances are changed.


**Use Shopify from Claude Code, Codex, Grok Build and MCP clients through PipMCP. One PipMCP API key connects this package to private execution.**

[Activate with PipMCP](https://pipmcp.com/github/start?integration=shopify-mcp&app=shopify&utm_source=github&utm_campaign=github-shopify-mcp) · [Integration directory](https://jannikco.github.io/pipmcp-connectors/integrations/shopify/)

## Build something useful

Inspect store products, then explicitly approve a title change to one chosen product.

The package is an authenticated client: discovery, schemas, policy decisions and execution come from PipMCP. Provider implementation remains on the server.

## Quick start

Requires Node.js 22+. Clone this repository, then:

```sh
npm ci --ignore-scripts
npm test
npm start
# After configuring your provider in PipMCP and setting PIPMCP_API_KEY:
npm start -- --live
# Launch the stdio MCP bridge:
npm run server
```

Create a PipMCP API key with explicit connectors:read and connectors:call scopes. Set PIPMCP_API_KEY in the calling process environment. Provider credentials are configured in PipMCP. Multiple accounts require explicit PIPMCP_INTEGRATION_ID and PIPMCP_CONNECTION_ID from discovery. Existing keys do not gain scopes automatically.

## Built for coding agents

Install this repository's skill into your project:

```sh
node lib/cli.mjs agents install --client codex --scope project --project /path/to/project
# Use claude or grok for those clients.
```

Connect AI agents to Shopify using one PipMCP API key and actual hosted tool discovery. Native AGENTS.md, CLAUDE.md and project skills support the current coding session. No additional AI model key is required. [capabilities.json](capabilities.json) describes the package; [llms.txt](llms.txt) indexes its instructions.

## Inspect actual tools before calling

```js
import {PipMCPGatewayClient} from './lib/index.mjs';
const client = new PipMCPGatewayClient(); // PIPMCP_API_KEY
const inventory = await client.getTools("shopify");
console.log(inventory.data.tools);
// Select a real tool and validate your arguments against its returned schema.
// client.callTool(provider, actualToolName, reviewedArguments, {idempotencyKey});
```

PipMCP enforces workspace, role, connection, policy/approval and usage controls. Preserve the original idempotency key when retrieving/resuming a receipt. Pending and confirmation_required are not completed provider results; uncertain writes are held for reconciliation. The bridge never auto-approves a hosted action.

## Version 1 migration

This release uses the authenticated PipMCP gateway for real discovery/execution. Upgrade your MCP launch command and configure PIPMCP_API_KEY before reconnecting. Historical local runtimes remain available under their original MIT license in Git history. 

## Evidence and support

Transport, package and agent fixtures are tested. Authentication/provider access must be checked in your own configured workspace; synthetic fixtures do not establish live provider execution. Do not include credentials or customer records in public issues. MIT covers this public client.

## Provider setup reference

[Official provider documentation](https://shopify.dev/docs/api/admin-graphql/2026-10/queries/products). Credentials are entered in PipMCP; this package does not accept or store provider secrets.
