# Existing tally-mcp-server feature catalogue

Baseline: local Claudally commit `74869e5388da3ba5c30fbd01ef72af4ca32d38fd`, version 0.10.0. This catalogue describes source registrations; runtime availability and Copilot discovery remain UNVERIFIED.

The server currently registers 56 MCP tools (55 in `src/mcp.mts`, plus `feeding-guide`) and two prompts (`feed-tally-data`, `profile-client-books`). It supports local stdio and remote Streamable HTTP. The broad product contains read and write capabilities; this catalogue does not authorize executing writes.

## Read and reporting tools

`list-master`, `search-master`, `chart-of-accounts`, `trial-balance`, `profit-loss`, `balance-sheet`, `stock-summary`, `ledger-balance`, `stock-item-balance`, `bills-outstanding`, `party-details`, `locate-voucher`, `find-voucher-by-reference`, `ledger-account`, `stock-item-account`, `gst-voucher-details`, `stock-item-gst`, `gst-hsn-summary`, `gstr1-summary`, and `gstr2-summary` retrieve Tally masters, ledgers, balances, inventory, parties, vouchers, GST details and report-style summaries.

These are fixed report/template-backed operations, not proof of complete coverage of every Tally module or build. Results require company/date reconciliation. `chart-of-accounts` reads Tally’s hierarchy; it does not load or validate an external Chart of Accounts workbook.

## Company and operational tools

`list-companies` scans configured Tally data folders; `list-available-companies` discovers on-disk metadata; `list-loaded-companies` checks currently accessible companies; `list-configured-companies` returns aliases and credential-presence metadata; `resolve-company` resolves an exact name, folder or alias; `set-active-company` changes the process active-company pointer; `use-company`, `open-company`, `load-company`, `load-company-by-alias`, `switch-company`, `open-tally`, and `unlock-stored-credentials` can select, load, restart, launch or unlock Tally through configuration or GUI paths.

`status`, `get-context`, `get-period`, and `open-company-debug` report health, environment, period or troubleshooting details. `set-entry-order` persists voucher-entry ordering. These tools are not employee authorization and can expose or change operational state. They are excluded from the first review-only evaluation.

## Write tools

`create-voucher`, `create-vouchers`, `create-gst-voucher`, `alter-voucher`, `reverse-voucher`, `delete-voucher`, `delete-vouchers`, `gui-delete-voucher`, `create-ledger`, `create-stock-item`, `set-ledger-gst`, and `update-party-details` create, alter, cancel, delete or configure Tally records. Dry-run/confirmation behavior is tool-specific and is not a universal approval mechanism. Test writes only later on a disposable company under a separate approval.

## Debug, SQL and GUI tools

`query-database` queries temporary DuckDB report tables; `tally-raw-xml-probe` submits raw XML under debug gates; `gui-screenshot` captures the Tally window; `gui-send-keys` sends UI input; `repair-master-names` can rename malformed masters. `feeding-guide` provides bookkeeping guidance. SQL, raw XML, GUI and repair tools are outside the first evaluation.

## Existing security/platform behavior

The HTTP server binds to loopback by default, requires a startup password, implements OAuth-style authorization with PKCE, bearer/refresh tokens, discovery and registration, and includes rate limits/backoff, Host/CORS checks and read-only guards. Tally XML uses fixed templates and bounded socket timeouts. Windows installer/service and GUI companion scripts are present.

These protections do not yet provide employee-to-company authorization, per-session company isolation, owner-scoped report downloads, or a client module policy. The broad source also contains global active-company state and write/GUI/process paths. The future client profile must enforce allowlists in the backend, not prompts or hidden tool descriptions.

## First client profile

Enable only company reads, Day Book reads, deterministic review, and authorized review reports. Disable writes, raw XML, SQL, GUI, company loading/restart and unrelated reports. The review engine and Chart of Accounts/rule loading still need to be implemented in this repository; the existing server does not provide the full requested Tally Review Agent out of the box.
