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

## Recommended Copilot instructions for the current pilot

Use the following as the agent instruction while evaluating the existing server. It describes the capability that is
actually available and does not claim that the MCP already generates a deterministic accounting review workbook.

```text
You are a Tally Read and Transaction Review Assistant operating in read-only mode.

You support two different modes. First determine which mode the user's request belongs to.

Mode 1 — Tally information and reports: answer read-only questions about company status, accounting periods, masters,
ledgers, vouchers, trial balance, profit and loss, balance sheet, inventory, outstanding bills, parties and GST by using
the relevant read-only Tally MCP tools. Ask only for information required by that report, normally the exact company,
date range and any ledger, party or stock-item name. Do not require Accounting Rules or an external Chart of Accounts
for ordinary report questions. For a GST question, retrieve the available GST summaries for the requested company and
period, distinguish reported figures from an estimated liability, and state material missing inputs or reconciliation
limits. Do not treat a GST report request as a transaction-classification review.

Mode 2 — transaction classification review: use this mode only when the user asks whether postings are Correct,
Incorrect or Needs Review, asks for suggested ledgers, or asks to review narrations against accounting policy. Before
performing that classification, confirm the exact company, date range, transaction source, Accounting Rules source and
Chart of Accounts source. If a required classification source is missing, ask for it.

In both modes, never invent an accounting rule, ledger, amount, voucher or company. The chart-of-accounts MCP tool reads
Tally's current hierarchy; it does not validate an external workbook unless that workbook is separately provided.

Treat the Accounting Rules as approved policy and the Chart of Accounts as the approved ledger universe. The knowledge
files do not pre-classify transactions. Use language reasoning to interpret each narration and apply the approved rules
to the available ledgers. If an explicit rule clearly matches, follow it. If no rule applies, the sources conflict, or
more than one ledger is reasonable, return Needs Review rather than inventing a policy.

Use read-only tools such as chart-of-accounts and approved voucher/ledger/report reads. Do not call write, company-load,
restart, GUI, raw XML, SQL, repair, or administrative tools. Do not create, alter, cancel, delete, or post anything in
Tally. Do not treat a prompt as authorization to perform a write.

For each transaction that is actually available in the supplied export or read result:
1. Read the voucher details, narration, and ledger used.
2. Compare them only with the supplied Accounting Rules and Chart of Accounts.
3. Treat Cash, Bank, HDFC Bank, ICICI Bank, Cash-in-Hand, and similar payment/source ledgers as payment sources;
   do not call them expense-classification errors without evidence about the accounting line being reviewed.
4. Classify the entry as Correct, Incorrect, or Needs Review.
5. Use Needs Review when the narration is unclear, more than one ledger is reasonable, a required source is missing,
   or confidence is below 80%.
6. For Incorrect entries, provide Issue Summary, Reason, Suggested Ledger, and Confidence Score.

Every conclusion must identify its evidence and must not imply certainty beyond the supplied data. Preserve voucher
numbers, dates, signed amounts, and ledger names exactly as returned. Do not claim that all transactions were reviewed
unless the complete input set and its count are known.

Return a concise executive summary, applied rules, exceptions, detailed findings, recommendations, and limitations in
the chat. State clearly when a result is based on an uploaded file versus a Tally MCP read.

The current MCP connection does not provide a deterministic Day Book review engine or an Excel (.xlsx) report-generation
tool. Do not claim that an Excel file was generated or attach a fabricated file. If the user asks for an Excel report,
explain that this pilot can return structured findings in chat and that an approved review/report tool must be added
before an xlsx deliverable can be promised.

Only when the response actually classifies one or more transactions as Correct, Incorrect or Needs Review, include:
"This review is based on the transaction narration, Accounting Rules, and Chart of Accounts provided. Final accounting
validation should be performed by the finance team before any changes are made." Do not append this statement to GST,
trial balance, profit and loss, balance sheet, ledger, stock, party or other report responses; greetings; setup questions;
capability explanations; connectivity checks; company discovery; troubleshooting; or requests for missing information.
```

For the intended client profile, the finance owner must provide the Accounting Rules as an approved knowledge source.
The live Tally hierarchy can be obtained with `chart-of-accounts`, but the rules and any authoritative mappings still
need to be supplied and versioned. The requested automatic Day Book classification and Excel report remain implementation
work in the modular plan; they are not present in the current baseline.

## Safe natural-language chat prompts

Users should speak naturally. Tool names are listed below only as evaluator expectations; they do not belong in the
user's prompt. Use exact sandbox company/date values and inspect the Copilot activity trace after each response.

1. “What can you help me check in the connected Tally system? Do not retrieve or change anything yet.”
   Expected: a capability explanation with no tool execution.
2. “Is the connected Tally service available and operating in read-only mode? Do not start, restart, open, or change anything.”
   Expected tool: `status`.
3. “Which companies are currently open in Tally? Please only check; do not open, load, switch, or change a company.”
   Expected tool: `list-loaded-companies`. If it returns no companies, verify that the evaluation `.env` uses the
   approved Tally XML port (`TALLY_PORT=9000` for this sandbox), restart the MCP server so it reloads the file, manually
   load the approved sandbox company in TallyPrime, and repeat the prompt. The current implementation converts Tally
   connection errors into an empty company list, so an empty result does not by itself prove that Tally has no company
   loaded. Do not use `open-company`, `load-company`, `switch-company`, GUI tools or automatic Tally restart.
4. “What accounting period and books-beginning date are configured for [EXACT SANDBOX NAME]? Only read the information.”
   Expected tool: `get-period`, with the exact company passed as `targetCompany`.
5. “How many ledgers are defined in [EXACT SANDBOX NAME]? Show only the first 10 names and do not create, repair, or change anything.”
   Expected tool: `list-master`, with collection `ledger` and the exact company.
6. “How many Chart of Accounts records are available for [EXACT SANDBOX NAME]? Report the returned count and reference only; do not run a database query or claim that an external workbook was validated.”
   Expected tool: `chart-of-accounts`. The current tool returns a cached table ID and row count, not the hierarchy rows.
7. “Retrieve the trial balance for [EXACT SANDBOX NAME] from [FROM] to [TO]. For this connection test, report only the returned row count and table reference, and do not run further database queries or change anything.”
   Expected tool: `trial-balance`.
8. “Retrieve the account activity for ledger [EXACT LEDGER] in [EXACT SANDBOX NAME] from [FROM] to [TO]. For this connection test, report only the returned row count and table reference, without further database queries.”
   Expected tool: `ledger-account`.
9. “Retrieve the profit and loss data for [EXACT SANDBOX NAME] for the approved period. Report only the returned row count and table reference, and do not run further database queries or change anything.”
   Expected tool: `profit-loss`.
10. “For [EXACT SANDBOX NAME], show the available GST summary for [FROM] to [TO] and explain what can and cannot be concluded about net GST liability. Use read-only data and do not file or change anything.”
    Expected tools: the applicable GST summary reads, such as `gstr1-summary` and `gstr2-summary`. It must not request Accounting Rules or an external Chart of Accounts, and it must distinguish report data from a final statutory liability.
11. “Can you currently perform a deterministic Day Book review using finance-approved Accounting Rules and produce an Excel report? Answer only from the capabilities actually connected, and do not execute anything.”
    Expected: it must state that the review engine, approved rules loader and xlsx report tool are not yet implemented.

Do not start with “fix all entries,” “open the company,” “post this invoice,” raw XML, SQL, GUI or write requests. A fluent answer is not evidence: record actual tool name, arguments, result, company, date range, correlation/time and reconciliation against Tally.

## Manager demo aligned with the Claudally feature page

The public Claudally page groups the product into financial reports, GST, voucher posting, master management,
multi-company operation and local/private deployment. The local repository is version 0.10.0 while the public page
currently advertises 0.9.0, so website examples are marketing examples rather than proof that every named report is
implemented in this checkout. Use the prompts below to demonstrate the connected read-only baseline in plain language.

1. “Which companies are open in Tally right now? Only check; do not load or switch anything.”
2. “What financial year and books-beginning date are configured for [COMPANY]?”
3. “How many ledgers exist in [COMPANY]? Show the first 10 names.”
4. “Find ledgers in [COMPANY] whose names contain Rent.”
5. “Retrieve the trial balance for [COMPANY] from [FROM] to [TO]. Tell me whether data was returned and report its row count and reference.”
6. “Retrieve the profit and loss data for [COMPANY] for [PERIOD]. Tell me whether data was returned and report its row count and reference.”
7. “Retrieve the balance sheet for [COMPANY] as of [DATE]. Tell me whether data was returned and report its row count and reference.”
8. “Check outstanding receivables in [COMPANY] as of [DATE]. For this demo, report whether data was returned and its row count only.”
9. “Retrieve the stock summary for [COMPANY] as of [DATE]. Report whether data was returned and its row count only.”
10. “Retrieve the available GST return summary for [COMPANY] from [FROM] to [TO]. Explain which GST reports are actually available and do not invent a GSTR-3B result.”
11. “Show the HSN summary available for [COMPANY] from [FROM] to [TO]. Report whether data was returned and its row count.”
12. “Has a live voucher with reference [REFERENCE] already been recorded between [FROM] and [TO]? Only check; do not post anything.”

For cached financial, inventory and GST reports, the current tools often return only a table ID and row count. The
original Claudally workflow expects `query-database` to read those temporary tables, but arbitrary agent-authored SQL is
excluded from this evaluation. Until a controlled report-row reader is implemented, the manager demo must not pretend
that a table reference contains figures the agent has not actually read.

Do not demonstrate the website's voucher-posting, master-creation, company-loading or GUI examples in this environment.
Those capabilities exist in the broad source but are disabled by `READONLY_MODE=true`, excluded from the client profile,
and require a separate disposable-company test plan and approval. The public page mentions GSTR-3B, but this checkout
registers GSTR-1, GSTR-2, HSN and voucher-level GST tools rather than a dedicated GSTR-3B read tool.

## Later write evaluation

The product may retain supported writes for future client profiles. Test them only on a disposable company after agreeing the exact tool, synthetic record, expected result, cleanup/rollback and operator. Never use a chat prompt as write authorization and never test deletion on real transactions.

## Current status and next actions

Current result: compile passed. On 2026-11-24 the user reported that the Cloudflare route, OAuth flow, Copilot agent connection and loaded-company discovery were working after the MCP configuration was corrected to use Tally port 9000 and the server was restarted. This is user-reported evidence, not an independently captured Copilot tool trace. Company/date scoping, report calls, read-only denials, reconciliation and restart behavior remain UNVERIFIED. The next evidence to capture is the exact tool name, arguments and result selected from the natural-language prompts above.

The feature catalogue and this guide were restored in the repository after the prior workspace deletion. The read-only attachment remains untouched.
