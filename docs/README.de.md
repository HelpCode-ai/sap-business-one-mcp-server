# SAP Business One MCP Server

[English](../README.md) · **Deutsch**

**Verbinde SAP Business One mit Claude, ChatGPT und Copilot: geschäftspartner, Artikel, Verkaufsbelege, Eingangsrechnungen, Zahlungen und Buchungen als MCP-Tools.** Basiert auf [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp).

SAP Business One MCP Server gibt Claude, ChatGPT, Copilot und Cursor 24 Tools für SAP Business One: geschäftspartner, Artikel, Verkaufsbelege, Eingangsrechnungen, Zahlungen und Buchungen. 23 Tools lesen, 1 können Daten ändern. Es läuft auf AnythingMCP: mit einem Klick in AnythingMCP Cloud oder selbst gehostet mit Docker. Zugangsdaten werden verschlüsselt gespeichert, jeder Aufruf landet im Audit-Log.

**Zuletzt geprüft:** 2026-10-09 gegen a production SAP Business One company database on SQL Server, through AnythingMCP Cloud (read traffic on business partners, items, A/R and A/P invoices, payments, journal entries and the chart of accounts).  
**Adapter synchronisiert:** <!-- synced -->2026-10-09

Maintained by [helpcode.ai](https://helpcode.ai), the team that builds and maintains [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp).

## Schnellstart (AnythingMCP Cloud)

1. Melde dich bei [cloud.anythingmcp.com](https://cloud.anythingmcp.com) an und öffne den [Installationslink](https://cloud.anythingmcp.com/connectors/store?install=sap-business-one).
2. Trage `SAP_B1_HOST`, `SAP_B1_PORT`, `SAP_B1_COMPANY_DB`, `SAP_B1_USERNAME`, `SAP_B1_PASSWORD` ein (siehe [Authentifizierung](#authentifizierung)).
3. Kopiere die URL deines MCP-Servers unter **MCP Servers** und füge sie in deinen KI-Client ein ([siehe unten](#claude-chatgpt-copilot-oder-cursor-verbinden)).

AnythingMCP Cloud ist derselbe Open-Source-Code, betrieben von helpcode.ai in Frankfurt.

## Selbst gehostet (Docker)

Benötigt Docker 24+, openssl und Node 18+.

```bash
git clone https://github.com/HelpCode-ai/sap-business-one-mcp-server.git
cd sap-business-one-mcp-server
./scripts/install.sh
```

`install.sh` schreibt `.env` mit neuen Secrets, startet AnythingMCP, legt den ersten Admin an, installiert den Connector, sofern `SAP_B1_HOST` und `SAP_B1_PORT` und `SAP_B1_COMPANY_DB` und `SAP_B1_USERNAME` und `SAP_B1_PASSWORD` in `.env` gesetzt sind, und erzeugt einen MCP-API-Key. Ohne Zugangsdaten gibt es stattdessen den Installationslink aus: `http://localhost:3000/connectors/store?install=sap-business-one`. Danach die ganze Kette prüfen:

```bash
npm install && node scripts/smoke.mjs
```

## Claude, ChatGPT, Copilot oder Cursor verbinden

- **Claude (claude.ai, Desktop, Mobil):** *Customize → Connectors → Add custom connector*, MCP-Server-URL einfügen und anmelden. Claude verbindet sich aus der Cloud von Anthropic, die URL muss also öffentlich per HTTPS erreichbar sein: deine AnythingMCP-Cloud-URL oder deine eigene Instanz mit TLS.
- **Claude Code:**

  ```bash
  claude mcp add --transport http sap-business-one-mcp-server http://localhost:4000/mcp --header "X-API-Key: <MCP_API_KEY>"
  ```
- **Cursor** (`.cursor/mcp.json`) und **VS Code / GitHub Copilot** (`.vscode/mcp.json`, Schlüssel `servers` statt `mcpServers`, dazu `"type": "http"`):

  ```json
  { "mcpServers": { "sap-business-one-mcp-server": { "url": "http://localhost:4000/mcp", "headers": { "X-API-Key": "<MCP_API_KEY>" } } } }
  ```
- **ChatGPT:** die öffentliche HTTPS-URL in den ChatGPT-Einstellungen als Connector (App) hinzufügen. Eine `localhost`-URL funktioniert dort nicht.

## Tools

24 Tools, erzeugt aus [`adapter/sap-business-one.json`](../adapter/sap-business-one.json). Tools mit **lesen** können im Quellsystem nichts ändern.

<!-- tools:start (generated from adapter/*.json, do not edit) -->
| Tool | Funktion | Zugriff |
|---|---|---|
| `b1_list_business_partners` | List business partners (customers, suppliers, leads) with their code, name, type and balance. | lesen |
| `b1_get_business_partner` | Read one business partner by CardCode: addresses, contacts, payment terms and balance. | lesen |
| `b1_list_items` | List inventory items with their code, name, prices and stock per warehouse (ItemWarehouseInfoCollection). | lesen |
| `b1_get_item` | Read one item by ItemCode: prices, units, groups and stock in each warehouse. | lesen |
| `b1_list_orders` | List sales orders with their customer, dates, totals and status (bost_Open / bost_Close). | lesen |
| `b1_get_order` | Read one sales order by DocEntry (integer key, not DocNum), with all its lines. | lesen |
| `b1_create_order` | Create a new sales order. | schreiben |
| `b1_list_invoices` | List A/R invoices (sales invoices to customers). | lesen |
| `b1_get_invoice` | Read one A/R invoice by DocEntry (integer key, not DocNum), with lines and taxes. | lesen |
| `b1_list_quotations` | List sales quotations with their customer, validity date, totals and status. | lesen |
| `b1_list_delivery_notes` | List delivery notes (goods shipped to customers) with their customer, date and lines. | lesen |
| `b1_list_credit_notes` | List A/R credit memos (credit notes issued to customers) with their totals and status. | lesen |
| `b1_list_purchase_invoices` | List A/P invoices (supplier invoices). | lesen |
| `b1_get_purchase_invoice` | Read one A/P invoice in full, with its lines, taxes and withholding tax. | lesen |
| `b1_list_purchase_credit_notes` | List A/P credit memos (credit notes received from suppliers). | lesen |
| `b1_list_vendor_payments` | List outgoing payments to suppliers, with the invoices each one settles (PaymentInvoices). | lesen |
| `b1_list_incoming_payments` | List incoming payments from customers, with the invoices each one settles (PaymentInvoices). | lesen |
| `b1_list_journal_entries` | List journal entries with their lines (JournalEntryLines: account, debit, credit). | lesen |
| `b1_get_journal_entry` | Read one journal entry by JdtNum (TransId) with all its lines: account, debit and credit. | lesen |
| `b1_list_chart_of_accounts` | List G/L accounts of the chart of accounts with their code, name, type and balance. | lesen |
| `b1_list_bank_statements` | List imported bank statements with their account, date and balances. | lesen |
| `b1_list_external_reconciliations` | List external (bank) reconciliations of G/L or business partner accounts in a date or number range. | lesen |
| `b1_get_external_reconciliation` | Read one external (bank) reconciliation: amount, date, type and the journal entry and bank statement lines it matched. | lesen |
| `b1_get_company_info` | Sanity check: returns company metadata (admin info). | lesen |
<!-- tools:end -->

## Beispiel-Prompts

- Welche Kundenaufträge aus dieser Woche sind noch offen?
- Zeig mir Geschäftspartner C20000 mit Saldo und Kontaktdaten.
- Welche Ausgangsrechnungen sind mehr als 30 Tage überfällig?
- Welche Angebote aus dem letzten Monat wurden noch nicht zum Auftrag?
- Lege einen Auftrag für C20000 an: 10 Stück A00001. (schreibend)
- Welche Artikel hatten in den letzten 90 Tagen keinen Umsatz?

Weitere (auf Englisch) in [examples/prompts.md](../examples/prompts.md).

## Authentifizierung

**Setup**:
1. Identify your Service Layer host (on HANA it usually runs on the database server; SQL Server installs and hosting partners run it on a separate host). Default HTTPS port is `50000`.
2. Create or pick a B1 user that has API access for the company database you want to expose. A user without write authorizations is a safe start: every tool except `b1_create_order` only reads.
3. Find the `CompanyDB` name in **Choose Company** dialog of the B1 client (looks like `SBODEMOGB`).
4. Set the five env vars below. The adapter logs in once per session (~25 min TTL) and re-uses the `B1SESSION` cookie.

**OData**: this adapter targets the **v2 / OData v4** Service Layer endpoint at `/b1s/v2/`. Older `/b1s/v1/` (OData v3) deployments are not covered — upgrade to FP 2405+ first.

**Filtering**: use standard OData `$filter` syntax, e.g. `CardCode eq 'C20000'` or `DocDate ge '2026-01-01'`.

**Paging**: the Service Layer returns 20 rows per call unless the client asks for more, whatever `$top` says. Every list tool asks for pages of up to 100 rows (`Prefer: odata.maxpagesize=100`); when a response carries `@odata.nextLink`, call the same tool again with `skip` raised by the rows already read. Use `select` to keep wide documents (journal entries with their lines, invoices) small.

**Keys and enums**: documents are read by `DocEntry` (internal key), not by `DocNum` (the number printed on the document). Booleans are `tYES` / `tNO`, document status is `bost_Open` / `bost_Close`, and `OriginalJournal` tells what created a journal entry (`ttJournalEntry` for manual entries, `ttAPInvoice`, `ttARInvoice`, `ttVendorPayment`, `ttReceipt`…).

**Finance**: A/P invoices, A/P and A/R credit memos, outgoing and incoming payments, journal entries, the chart of accounts and bank statements each have a list tool. Bank reconciliations are not a list in the Service Layer: `b1_list_external_reconciliations` calls its reconciliation service (a POST that only reads) and returns account and number pairs for `b1_get_external_reconciliation`; open items are the documents whose `PaidToDate` is below `DocTotal`.

**Permissions**: a `403` with SAP error `-6006` ("Modifying this object is not permitted for current user") means the login works but the B1 user lacks the authorization for that object. Grant it in **Administration → System Initialization → Authorizations**, or keep the user read-only on purpose.

**Self-host caveat**: Business One is on-premise. The Service Layer is reachable from inside the customer network. To use the AnythingMCP Cloud, expose Service Layer via a reverse proxy + TLS cert; otherwise self-host the adapter on the same network.

## Sicherheit

- **Lesen oder schreiben entscheidest du.** 23 der Tools lesen nur; `b1_create_order` können Daten ändern. Weise den Connector einem MCP-Server zu, dessen Rolle nur die gewünschten Tools freigibt; die anderen sieht dieser Client gar nicht.
- **Zugangsdaten** werden mit AES-256-GCM verschlüsselt und nie an das Modell gegeben.
- **Response-Mapping** entfernt oder formt Felder pro Tool, bevor sie das Modell erreichen, etwa Bankdaten oder personenbezogene Daten.
- **Audit-Log:** Jeder Aufruf wird mit Eingabe, Ausgabe, Dauer und Status protokolliert, selbst gehostet in deiner eigenen Datenbank.
- **SSO, RBAC und SCIM** sind in der selbst gehosteten Version enthalten.

## FAQ

### Gibt es einen MCP-Server für SAP Business One?
Ja, diesen hier. Er verbindet den Service Layer von SAP Business One über AnythingMCP mit Claude, ChatGPT und Copilot: 12 Tools für Geschäftspartner, Artikel, Aufträge, Ausgangsrechnungen, Angebote und Lieferscheine, dazu das Anlegen von Aufträgen.

### Was brauche ich für die Verbindung?
Host und Port des Service Layers (meist 50000), den Namen der Firmendatenbank und einen B1-Benutzer mit API-Zugriff.

### Kann die KI Belege anlegen oder ändern?
Sie kann mit `b1_create_order` Aufträge anlegen; alles andere liest nur. Nimm das Tool aus der Rolle des MCP-Servers, wenn die KI nur lesen soll.

### Ist das mit einem echten SAP-B1-System getestet?
Noch nicht. Der Adapter folgt der Service-Layer-Dokumentation von SAP; Rückmeldungen sind willkommen.

## Fehlerbehebung

| Problem | Lösung |
|---|---|
| `401` / `403` vom Hersteller | Zugangsdaten falsch oder ohne Rechte. Auf der Connector-Seite neu eintragen; der Import macht einen Testaufruf und zeigt das Ergebnis. |
| Tools fehlen im KI-Client | Der Connector ist nicht dem MCP-Server zugewiesen, den der Client nutzt. **MCP Servers** prüfen, dann `node scripts/smoke.mjs` ausführen. |
| Das System steht im internen Netz | AnythingMCP in diesem Netz selbst hosten und den Hostnamen in `SSRF_ALLOWED_HOSTS` eintragen, sonst blockiert der Outbound-Guard den Aufruf. |
| Lokal ok, in AnythingMCP Cloud nicht | Das System muss aus dem Internet mit gültigem TLS-Zertifikat erreichbar sein. |

## Verwandte Repositories

- [sap-mcp-server](https://github.com/HelpCode-ai/sap-mcp-server): SAP MCP server: connect SAP Business One, S/4HANA (Cloud, on-premise via OData or HANA SQL) and Concur to Claude & ChatGPT.
- [odoo-mcp-server](https://github.com/keysersoft/odoo-mcp-server): Odoo MCP server: connect Odoo ERP to Claude & ChatGPT. Search, read, create and update any model: partners, orders, invoices.
- [weclapp-mcp-server](https://github.com/kochfreiburg/weclapp-mcp-server): weclapp MCP server: connect weclapp Cloud ERP to Claude & ChatGPT. Customers, orders, invoices, quotes and opportunities.
- [AnythingMCP](https://github.com/HelpCode-ai/anythingmcp): der Open-Source-MCP-Server und -Gateway, auf dem dieses Repository aufbaut.

## Lizenz

AGPL-3.0-only. Die Adapter-Definition in `adapter/` stammt aus AnythingMCP (AGPL-3.0).
