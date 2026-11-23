# Copilot evaluation guide

This is a test plan, not a deployment record. Current scope: this machine and an approved sandbox company such as Matrix3D Sandbox. No production Tally, write operation, service installation, DNS/firewall change or public endpoint change is authorized by this file.

## Intended flow

`Copilot Studio -> approved HTTPS hostname -> Cloudflare Tunnel -> 127.0.0.1:3000 tally-mcp-server -> Tally XML 9000`.

Use an organization-owned named Cloudflare Tunnel with a stable hostname for evaluation. Quick Tunnels are temporary, change hostname on restart and have transport limitations; do not treat one as production. Cloudflare is in the HTTP/TLS data path, so financial-data handling must be approved. The existing `tally.rahulporel.com` hostname is usable only after its route is verified; a public 502 does not prove it reaches this application.

## Safety gate

Before exposing an endpoint, confirm the exact test machine, sandbox company, approved hostname/account, test window and named testers. Use a separate evaluation configuration and OAuth state; never load the real `.env` for tests. Keep `READONLY_MODE` enabled, `TALLY_DEBUG_XML` and `ENABLE_GUI_CONTROL` disabled, bind MCP to loopback, and do not expose port 9000. Do not use prompts as a security control. If isolation cannot be demonstrated, stop at a fake Tally fixture.

## Local checks completed

On 2026-11-23, the installed TypeScript compiler completed `--noEmit` successfully using the available Node 24.22.3 runtime. Cloudflared is installed as version 2026.10.1 and its Windows service exists, but its token/route was not printed. Tally and cloudflared were running; no listener was found on MCP port 3000, while Tally listened on 9000. No tunnel was started and no Tally request was made.

## Copilot setup after local validation

In Copilot Studio: Tools -> Add a tool -> New tool -> Model Context Protocol. Use the approved HTTPS URL ending `/mcp`. Select OAuth 2.0 and validate discovery/registration or use a deliberately configured manual flow. Never paste the server password as a bearer token or into chat. Copilot supports Streamable HTTP; legacy `/sse` is not the target.

For the current Cloudflare Published Application, leave the optional **Path** field empty. A `/blog` value is only a dashboard placeholder and would prevent Copilot from reaching `/mcp`. An empty path forwards the MCP and OAuth paths to the backend, where authentication and tool authorization remain enforced. Preserve Host, Authorization, MCP session and protocol headers; disable caching. Narrow the published route only after the matcher has been tested against every required discovery and OAuth endpoint.

## Start an evaluation server

The repository runner creates `.env` from `.env.example` when it is missing. Prepare that local file with synthetic,
read-only settings before starting the HTTP server:

```powershell
Copy-Item .env.example .env
(Get-Content .env) `
  -replace '^PASSWORD=.*', 'PASSWORD=EvaluationOnly-ChangeMe-2026' `
  -replace '^ADMIN_SECRET=.*', 'ADMIN_SECRET=EvaluationAdminOnly-ChangeMe-2026' `
  -replace '^MCP_DOMAIN=.*', 'MCP_DOMAIN=https://tally.rahulporel.com' `
  -replace '^TALLY_PORT=.*', 'TALLY_PORT=19000' `
  -replace '^READONLY_MODE=.*', 'READONLY_MODE=true' `
  -replace '^ENABLE_GUI_CONTROL=.*', 'ENABLE_GUI_CONTROL=false' `
  | Set-Content .env -Encoding UTF8

npm run dev -- -NoAgent -NoTray
```

This uses the documented `scripts/dev-run-local.ps1` flow, builds the project, and starts only the MCP HTTP server and
TypeScript watcher. `TALLY_PORT=19000` deliberately avoids the live Tally listener on port 9000. Do not call a tool
that contacts Tally until an approved fixture or sanitized company is available. Stop it with `Ctrl+C`, or run
`npm run dev:stop` from another PowerShell window.

## Verify before Copilot

In a second PowerShell window:

```powershell
Invoke-WebRequest 'http://127.0.0.1:3000/.well-known/oauth-protected-resource' -UseBasicParsing
Invoke-WebRequest 'http://127.0.0.1:3000/.well-known/oauth-authorization-server' -UseBasicParsing
```

Then use `https://tally.rahulporel.com/mcp` in Copilot Studio. If either local request fails, stop and fix the local
server before testing the public hostname. Never put either synthetic secret in Copilot chat.

## Cloudflare challenge during Copilot login

If Copilot reports `RegisterDynamicClientAsync failed` and the response is a Cloudflare `Just a moment...` HTML page,
the edge challenge is blocking the server-to-server `POST /register`. Copilot cannot complete a JavaScript browser
challenge. Add a narrowly scoped Cloudflare WAF Skip rule for this hostname and the MCP/OAuth paths:

```text
(http.host eq "tally.rahulporel.com" and
 http.request.uri.path in {"/mcp" "/register" "/authorize" "/token"
 "/.well-known/oauth-protected-resource"
 "/.well-known/oauth-protected-resource/mcp"
 "/.well-known/oauth-authorization-server"})
```

Skip only the Cloudflare managed challenge/Bot or browser-check component that is producing the interstitial. Keep the
backend OAuth validation, password, rate limits and read-only policy enabled. Do not create a global hostname bypass or
paste `.env` contents into Cloudflare. After saving the rule, retry the Copilot connection; the response from
`/register` should be JSON, not an HTML challenge page.

If the response still has HTTP `403` and `Cf-Mitigated: challenge`, inspect Cloudflare Security Events for the matching
rule. On plans where zone-level Bot Fight Mode cannot be skipped by a custom rule, turn that feature off for this
evaluation zone or use a dedicated evaluation zone/hostname without the challenge. Do not disable the MCP server's own
OAuth checks; the Cloudflare edge must simply pass these machine-to-machine requests through.

## Safe chat prompts

Use exact sandbox company/date values and inspect the Copilot tool trace:

1. “List the available tally-mcp-server tools without executing them. Group them into reads, writes and administration.”
2. “Use status to report reachability and read-only mode. Do not start, restart or interact with Tally.”
3. “Use list-loaded-companies. Do not open, load or switch a company.”
4. “Use get-period for targetCompany [EXACT SANDBOX NAME]. Show books-from and financial-year dates.”
5. “Use list-master with collection ledger and targetCompany [EXACT SANDBOX NAME]. Return a count and small sample; do not create or repair anything.”
6. “Use chart-of-accounts for [EXACT SANDBOX NAME]. Summarize the returned hierarchy; do not claim it was checked against an Excel workbook.”
7. “Use trial-balance for [EXACT SANDBOX NAME] and [FROM] to [TO]. Preserve signed balances and do not change anything.”
8. “Use ledger-account for [EXACT LEDGER] in [EXACT SANDBOX NAME] and [FROM] to [TO]. Return representative voucher evidence only.”
9. “Use profit-loss for [EXACT SANDBOX NAME] and the approved period. Explain only returned figures.”
10. “Can this connected server already perform our deterministic Day Book review against our rules and Chart of Accounts and generate the Excel report? Answer only from available tools.”

Do not start with “fix all entries,” “open the company,” “post this invoice,” raw XML, SQL, GUI or write requests. A fluent answer is not evidence: record actual tool name, arguments, result, company, date range, correlation/time and reconciliation against Tally.

## Later write evaluation

The product may retain supported writes for future client profiles. Test them only on a disposable company after agreeing the exact tool, synthetic record, expected result, cleanup/rollback and operator. Never use a chat prompt as write authorization and never test deletion on real transactions.

## Current status and next actions

Current result: compile passed; tunnel route, MCP startup, OAuth handshake and Copilot invocation are UNVERIFIED. The existing Cloudflared Windows service is present and invokes `tunnel run`; no token, config path or route was printed. No direct `--url` origin was detected. Next, identify the route from the protected service configuration without revealing credentials, prepare an isolated runtime configuration, start the server only after confirming it cannot use real Tally, validate local MCP/OAuth, then connect a separate Copilot test connection.

The feature catalogue and this guide were restored in the repository after the prior workspace deletion. The read-only attachment remains untouched.
