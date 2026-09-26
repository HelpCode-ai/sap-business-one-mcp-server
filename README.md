# SAP Business One MCP Server

**English** · [Deutsch](docs/README.de.md)

**Connect SAP Business One to Claude, ChatGPT and Copilot: business partners, items, sales orders, invoices, quotations and deliveries as MCP tools.** Powered by [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp).

SAP Business One MCP Server gives Claude, ChatGPT, Copilot and Cursor 12 tools for SAP Business One: business partners, items, sales orders, invoices, quotations and deliveries. 11 tools read and 1 can change data. It runs on AnythingMCP: one click on AnythingMCP Cloud, or self-hosted with Docker. Credentials are stored encrypted and every call is audited.

**Status:** not yet verified against a live system. The adapter follows the vendor's API documentation; please report what you find.  
**Adapter synced:** <!-- synced -->2026-09-26

Maintained by [helpcode.ai](https://helpcode.ai), the team that builds and maintains [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp).

## Quick start (AnythingMCP Cloud)

1. Sign in at [cloud.anythingmcp.com](https://cloud.anythingmcp.com) and open the [install link](https://cloud.anythingmcp.com/connectors/store?install=sap-business-one).
2. Enter `SAP_B1_HOST`, `SAP_B1_PORT`, `SAP_B1_COMPANY_DB`, `SAP_B1_USERNAME`, `SAP_B1_PASSWORD` (see [Authentication](#authentication)).
3. Copy the URL of your MCP server under **MCP Servers** and add it to your AI client ([below](#connect-claude-chatgpt-copilot-or-cursor)).

AnythingMCP Cloud is the same open-source code, operated by helpcode.ai in Frankfurt, Germany.

## Self-hosted (Docker)

Needs Docker 24+, openssl and Node 18+.

```bash
git clone https://github.com/HelpCode-ai/sap-business-one-mcp-server.git
cd sap-business-one-mcp-server
./scripts/install.sh
```

`install.sh` writes `.env` with fresh secrets, starts AnythingMCP, creates the first admin, installs the connector if `SAP_B1_HOST` and `SAP_B1_PORT` and `SAP_B1_COMPANY_DB` and `SAP_B1_USERNAME` and `SAP_B1_PASSWORD` are set in `.env` and creates an MCP API key. Without credentials it prints the install link instead: `http://localhost:3000/connectors/store?install=sap-business-one`. Then check the whole chain:

```bash
npm install && node scripts/smoke.mjs
```

## Connect Claude, ChatGPT, Copilot or Cursor

- **Claude (claude.ai, Desktop, mobile):** *Customize → Connectors → Add custom connector*, paste your MCP server URL and sign in. Claude connects from Anthropic's cloud, so the URL must be public HTTPS: your AnythingMCP Cloud URL, or your own instance behind TLS.
- **Claude Code:**

  ```bash
  claude mcp add --transport http sap-business-one-mcp-server http://localhost:4000/mcp --header "X-API-Key: <MCP_API_KEY>"
  ```
- **Cursor** (`.cursor/mcp.json`) and **VS Code / GitHub Copilot** (`.vscode/mcp.json`, key `servers` instead of `mcpServers`, plus `"type": "http"`):

  ```json
  { "mcpServers": { "sap-business-one-mcp-server": { "url": "http://localhost:4000/mcp", "headers": { "X-API-Key": "<MCP_API_KEY>" } } } }
  ```
- **ChatGPT:** add the public HTTPS URL as a connector (app) in ChatGPT's settings. A `localhost` URL does not work there.

## Tools

12 tools, generated from [`adapter/sap-business-one.json`](adapter/sap-business-one.json). **read** tools cannot change anything in the source system.

<!-- tools:start (generated from adapter/*.json, do not edit) -->
| Tool | What it does | Access |
|---|---|---|
| `b1_list_business_partners` | List business partners (customers, suppliers, leads). | read |
| `b1_get_business_partner` | Get one business partner by CardCode. | read |
| `b1_list_items` | List inventory items (articles). | read |
| `b1_get_item` | Get one item by ItemCode. | read |
| `b1_list_orders` | List sales orders. | read |
| `b1_get_order` | Get one sales order by DocEntry (integer primary key). | read |
| `b1_create_order` | Create a new sales order. | write |
| `b1_list_invoices` | List A/R invoices. | read |
| `b1_get_invoice` | Get one A/R invoice by DocEntry. | read |
| `b1_list_quotations` | List sales quotations. | read |
| `b1_list_delivery_notes` | List delivery notes (outgoing goods movements). | read |
| `b1_get_company_info` | Sanity check: returns company metadata (admin info). | read |
<!-- tools:end -->

## Example prompts

- Which sales orders from this week are still open, and for which customers?
- Show business partner C20000 with their balance and contact details.
- What is the stock of item A00001 in each warehouse?
- Which A/R invoices are overdue by more than 30 days?
- Which quotations did we send last month that have not turned into orders?
- What did we deliver to customer C20000 this month?

More in [examples/prompts.md](examples/prompts.md).

## Authentication

**Setup**:
1. Identify your Service Layer host (typically the same machine that runs HANA + SAP B1). Default HTTPS port is `50000`.
2. Create or pick a B1 user that has API access for the company database you want to expose.
3. Find the `CompanyDB` name in **Choose Company** dialog of the B1 client (looks like `SBODEMOGB`).
4. Set the five env vars below. The adapter logs in once per session (~25 min TTL) and re-uses the `B1SESSION` cookie.

**OData**: this adapter targets the **v2 / OData v4** Service Layer endpoint at `/b1s/v2/`. Older `/b1s/v1/` (OData v3) deployments are not covered — upgrade to FP 2405+ first.

**Filtering**: use standard OData `$filter` syntax, e.g. `CardCode eq 'C20000'` or `DocDate ge '2026-01-01'`.

**Self-host caveat**: Business One is on-premise. The Service Layer is reachable from inside the customer network. To use the AnythingMCP Cloud, expose Service Layer via a reverse proxy + TLS cert; otherwise self-host the adapter on the same network.

## Security

- **Read or write is your choice.** 11 of the tools only read; `b1_create_order` can change data. Assign the connector to an MCP server whose role whitelists only the tools you want, and the rest are invisible to that client.
- **Credentials** are encrypted with AES-256-GCM and never shown to the model.
- **Response mapping** drops or reshapes fields per tool before they reach the model, e.g. bank details or personal data.
- **Audit log:** every call is recorded with input, output, duration and status, in your own database when self-hosted.
- **SSO, RBAC and SCIM** are included in the self-hosted build.

## FAQ

### Is there an SAP Business One MCP server?
Yes, this one. It connects the SAP Business One Service Layer to Claude, ChatGPT and Copilot through AnythingMCP: 12 tools for business partners, items, sales orders, A/R invoices, quotations and delivery notes, plus creating a sales order.

### What do I need to connect it?
The Service Layer host and port (usually 50000), the company database name and a B1 user with API access. The connector logs in and reuses the session cookie.

### Can the AI create or change documents in SAP B1?
It can create sales orders with `b1_create_order`; everything else only reads. Leave that tool off the MCP server's role if the AI should only read.

### Does it work with SAP B1 on HANA and on SQL Server?
It talks to the Service Layer, which ships with SAP Business One on HANA. Installations without a Service Layer can be connected through their SQL database instead, read-only.

### Has this been tested against a live SAP B1 system?
Not yet. The adapter follows SAP's Service Layer documentation; please report what you find.

## Troubleshooting

| Problem | Fix |
|---|---|
| `401` / `403` from the vendor | The credentials are wrong or lack rights. Re-enter them on the connector page; the import runs a test call and shows the result. |
| Tools missing in the AI client | The connector is not assigned to the MCP server the client uses. Check **MCP Servers**, then run `node scripts/smoke.mjs`. |
| The host is on your internal network | Self-host AnythingMCP on that network and add the hostname to `SSRF_ALLOWED_HOSTS`, or the outbound guard blocks the call. |
| Works locally, fails on AnythingMCP Cloud | The system must be reachable from the internet with a valid TLS certificate. |

## Related

- [sap-mcp-server](https://github.com/HelpCode-ai/sap-mcp-server): SAP MCP server: connect SAP Business One, S/4HANA Cloud and Concur to Claude & ChatGPT. Orders, partners, invoices as AI tools.
- [odoo-mcp-server](https://github.com/keysersoft/odoo-mcp-server): Odoo MCP server: connect Odoo ERP to Claude & ChatGPT. Search, read, create and update any model: partners, orders, invoices.
- [weclapp-mcp-server](https://github.com/kochfreiburg/weclapp-mcp-server): weclapp MCP server: connect weclapp Cloud ERP to Claude & ChatGPT. Customers, orders, invoices, quotes and opportunities.
- [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp): the open-source MCP server and gateway this repository is built on.

## License

AGPL-3.0-only. The adapter definition in `adapter/` comes from AnythingMCP (AGPL-3.0).
