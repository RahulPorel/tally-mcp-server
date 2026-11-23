# TallyPrime AI Integration — Codex Project Handoff

Date: 23 November 2026  
Project goal: Evaluate Claudally and the internal Tally Review Agent MVP, converge on a safe, maintainable Tally integration, and validate it against the company's Tally environment before any client rollout.

## 0. Critical operating rule

Analysis and local development are authorized objectives. Production changes are NOT automatically authorized. First inspect code, produce a written plan and gate checklist, and request explicit human approval before installing, publishing, opening NSG/firewall ports, changing DNS, creating service accounts, altering Tally settings, or running tests against the production Tally host. Treat all production Tally data as confidential. Initially perform READ-ONLY calls and preferably use `Matrix3D Sandbox` or a cloned/sanitized company. Do not execute voucher creation, alteration, deletion, data import, raw XML sent by an LLM, GUI keystrokes, arbitrary SQL or unapproved database access. Do not store credentials or financial data in Git or AI-facing logs.

## 1. Context / decisions from planning conversation

- Final deployment target for first in-company test: existing **Windows Server 2022** host running TallyPrime. Customer reference VM specs: **4 vCPU, 14 GB RAM, 150 GB SSD**; the company's own test VM may differ and must be discovered.
- First target is the **company's Tally environment**; client deployment comes only after end-to-end validation and management/IT approval.
- Existing Microsoft **Copilot Studio** agent remains in an organization-owned Microsoft tenant/environment. For clients, import a managed Power Platform solution and establish separate client-owned credentials and connections. Copilot Studio itself is a cloud service.
- Selected low-additional-cost architecture: **Windows-native MCP service + Caddy reverse proxy on the SAME Windows VM**; avoid new VM, Docker Desktop on Windows Server, costly Azure Application Gateway WAF v2/API Management for the pilot.
- Proposed data flow: `Copilot Studio -> HTTPS 443 (customer/company subdomain) -> Caddy -> localhost:3000 MCP -> localhost:9000 Tally XML API`.
- Externally exposed public endpoint should be an authenticated HTTPS MCP URL, normally `/mcp` (Streamable HTTP), NOT legacy `/sse`. The MCP server's own auth and authorization are mandatory; Caddy is routing/TLS, not an OAuth authorization service or WAF.
- Azure NSG + Windows Firewall: allow only carefully approved inbound TCP 443 to the Windows host (plus certificate-challenge traffic only if needed). Never expose TCP 3000, 9000 or RDP to the public internet. No public IP or DNS configuration should be made without approval.
- DNS: stable customer/company-owned subdomain, for example `tally-api.<approved-domain>`, DNS A record to a reserved static Azure Public IP if using direct ingress. Use a trusted TLS certificate with tested renewal. If existing VM is private and internet ingress is forbidden, report direct Path A as BLOCKED; offer an approved outbound/private alternative, not a workaround bypassing security policy.
- This is NOT '100% guaranteed for any client network'. Network, license, security and service integrations must pass client-specific compatibility gates. Additional cost may remain for public IP, DNS, licensing and monitoring.
- Initial production rollout: read-only report/data retrieval, not posting vouchers.

## 2. Claudally candidate repository

- Source: https://github.com/JINA-CODE-SYSTEMS/tally-mcp-server
- Site: https://claudally.jinacode.systems/
- Inspect current HEAD, pin commit SHA and dependency versions; do not rely on a changing `latest` image/installer.
- Repo claims Tally XML integration, ~29 tools, local stdio, remote Streamable HTTP, OAuth 2.1/PKCE, DuckDB analytics, Windows installer/Windows service, company-management tools, read-only mode.
- Crucial: licensed **AGPL-3.0-or-later** with additional attribution notice: https://github.com/JINA-CODE-SYSTEMS/tally-mcp-server/blob/main/NOTICE . Do not assume a proprietary derivative may be shipped without AGPL source/attribution obligations. Seek legal review or a separate commercial license. Original MIT upstream mentioned in NOTICE does not make Jina's changes MIT.
- Repo documentation must be reconciled: contemporary Windows installer may be *local-only*; separately documented `scripts/setup-windows.ps1 -DeploymentMode remote` and older Cloudflare remote workflow should NOT be assumed turnkey production. Claudally's own Cloudflare provisioning doc is flagged legacy/frozen. Verify current implementation and tests rather than trusting README marketing claims.
- Review risks: unrestricted raw XML or SQL, GUI automation/keystrokes, company isolation, per-user identity propagation, dynamic OAuth client registration, secrets storage, SSE compatibility, Windows Session 0 user-session automation, rate limiting, prompt injection, audit-log scope, writes and cross-company access.
- Read source/security assessment and verify actual current fixes, not just checkmarks in docs.
- `package.json` was observed to specify Node `>=22`; verify current file and compatible Windows deployment and native DuckDB package before builds.

## 3. Existing internal MVP / Tally Review Agent

Historical project (as of Oct 26-27, 2026) — verify against current repo and Copilot Studio environment:
- Agent name: `Tally Review Agent` in Microsoft Copilot Studio.
- Business goal: review Tally vouchers against `AccountingRules.docx` and `ChartofAccounts.xlsx`, flag Correct / Incorrect / Needs Review, and produce an Excel report with evidence and suggestions. An earlier process relied on uploading a Day Book export.
- TallyPrime 7.1 XML/HTTP connection on `127.0.0.1:9000` was validated with Postman (company and voucher responses). Historical test companies: `Bhrama Enterprises` and `Matrix3D Sandbox`. These may not be the currently loaded company or on today's VM.
- Experimental direct XML Copilot custom connector behind temporary Cloudflare Quick Tunnel returned XML during connector test, but agent execution previously failed with `ConnectorOperationNotFound`. A later update said Tally integration was working; **the exact final implementation and its code were NOT independently verified**. Discover and test what is actually running.
- Existing project handoff (if available in your workspace): `TallyPrime-Transaction-Review-Agent-Handoff.md` and test-only OpenAPI `TallyPrime-Direct-XML-REST-API.json`.
- Previous proposed middleware: Node.js/TypeScript JSON REST API with `/api/health`, `/api/companies`, `/api/vouchers/search`; controlled XML templates, parsing/validation, deterministic review, Excel generation. The document explicitly stated this scaffold was **not confirmed implemented** as of Oct 26.
- The temporary `trycloudflare.com` URL and raw `text/xml` endpoint are development-only; do not reuse them for production.
- Exact internal MVP Git repository or local path is UNKNOWN to this handoff. Inspect the user's selected project/workspace, ask for the repo path only if it cannot be found. Do not invent source files.

## 4. Initial architecture to validate (Caddy path)

```text
Finance user, approved Microsoft identity
              |
              v
Microsoft Teams / Copilot Studio Agent (company tenant)
              |
              | Streamable HTTP MCP over HTTPS, user-scoped OAuth 2.0 where supported
              v
Approved company-owned DNS: https://tally-api.<domain>/mcp
              |
              | Azure public IP + NSG inbound TCP 443 (ONLY after approval)
              v
Caddy Windows service on existing Windows Server 2022
              |
              | localhost / loopback HTTP 3000; no direct external access
              v
Hardened Tally MCP Windows service
              |
              | HTTP XML via localhost 9000, controlled read-only operations
              v
TallyPrime with approved company loaded
```

### Security / correctness requirements

- Auth: Need a working OAuth 2.0 mechanism accepted by Copilot Studio and validated by the MCP server; distinguish app identity from actual end-user identity. Never imply Caddy automatically validates OAuth. Do not assume Claudally's password-based OAuth enforces per-employee Microsoft Entra RBAC.
- Authorization: map authorized user/service identity to company + tool allowlist; enforce in backend, not model prompt. Deny writes. Treat errors/empty datasets/closed companies as explicit failures, not successful empty reports.
- MCP: test initialize, tools/list, tools/call, session behavior, content types, streaming, timeouts, origin/trusted-proxy settings; expose required OAuth discovery/authorization routes in Caddy in addition to `/mcp` where needed. The actual paths must be discovered from source. Do not publish all routes wholesale.
- Security: Windows service least privilege, config ACLs/DPAPI or approved secret store, TLS and certificate renewal, rate/size limits, Windows Defender Firewall, Azure NSG, restricted RDP, audit logs without raw accounting payloads, approved patch/rollback plan.
- Prompt injection and financial write safety: no raw XML generation from LLM; no GUI control or raw SQL/SQL filesystem functions; pin and validate allowed read-only queries, strict XML parser, input validation, bounded results.
- Tally: port 9000 loopback only; verify Tally availability during unattended operation, not only when interactive RDP is open. Avoid restart/changes to live Tally service until IT approves.
- Financial results: corroborate same company and date range against TallyPrime, preserve debit/credit signs and voucher identity; avoid false positive review results.

## 5. Revised deployment checklist (target state, NOT instructions to run immediately)

1. Check host version, privileges, installed Tally version, backup/recovery, available resources and approved change window.
2. Verify Tally XML server on localhost:9000, company loaded and sanitized read-only report query.
3. Compare Claudally vs internal MVP: reuse/adapt/build decision, licenses, features, tests and security gaps.
4. Build a pinned, reproducible Node/TypeScript Windows-compatible release (no Docker Desktop prerequisite); use Windows-native service via NSSM or WinSW, with least-privilege account, restart recovery and log rotation.
5. Configure Tally host/port, read-only tool allowlist, authentication, company restriction, safe secrets and audit logging.
6. Test MCP locally on loopback: health (if implemented), initialize, tools/list, tools/call, negative/authorization cases, restart and Tally unavailable.
7. Install Caddy on same VM only after IT approval; configure HTTPS forwarding to localhost:3000, required auth/discovery routes, streaming and request/time limits. Use a company-owned hostname and certificate lifecycle.
8. Configure static public IP + DNS record + Azure NSG + Windows Firewall for 443; no public 3000/9000/RDP; validate externally and test certificate renewability.
9. Import/configure Copilot Studio agent in company tenant, connect `https://tally-api.<domain>/mcp` with supported OAuth flow and test tool discovery and user-specific authorization.
10. End-to-end read-only finance test, reconciliation, audit trail, monitoring/alerting, failure handling and rollback. Document issues and sign-off before proposing client rollouts.

## 6. What Codex should do (four gated phases)

**Phase 0 — Workspace discovery + forensic audit (no modifications to production):**
- Locate both repositories. Identify exact commit, tree, source entrypoints, Node/version/dependencies, services, HTTP transport, auth, tooling, parsing, tests and deployment scripts. Do not assume both are present.
- Produce a feature matrix: Claudally vs MVP vs target architecture; choose refactor/reuse/new based on quality and license constraints.
- Create threat model and list BLOCKERS/P0/P1/P2; document the Copilot `ConnectorOperationNotFound` history and current verified state.
- Output `docs/architecture/AS_IS.md`, `TO_BE.md`, `REPO_COMPARISON.md`, `SECURITY_REVIEW.md`, `DECISIONS.md`, `OPEN_QUESTIONS.md`.

**Phase 1 — Sandbox proof of concept:**
- Implement minimal read-only Tally company/list + one ledger/report query, version-pinned reproducible build, secure streamable MCP, automated tests, fake Tally server fixture and a Codex MCP inspector verification. Keep secrets out of source.
- Preserve any working deterministic financial-review logic from MVP; do not rebuild it without justification.
- Do not deploy publicly or touch production Tally.

**Phase 2 — Staging / safe company-environment validation:**
- With explicit approvals, validate on nonproduction/sanitized company under a managed Windows host. Windows service restart, loaded Tally company behavior, proxy in local-only test, test DNS/TLS (without production route changes), auth/roles, Copilot tool-calls and audit. Write a rollback plan.

**Phase 3 — Restricted company production pilot:**
- HUMAN APPROVAL GATE. Verify change ticket, backup, test company or strictly limited read-only account and data scope, snapshot/rollback, monitoring and named IT/finance approvers. Only then perform approved setup of existing company VM, Caddy/DNS/NSG, OAuth/Copilot. Tests are read-only and low-volume. Stop on permission gaps, untrusted cert, unauthorized company access, or unexpected financial mismatches.
- Document `PROD_PILOT_RUNBOOK.md`, `NETWORK_MATRIX.md`, `ROLLBACK.md`, `UAT_RESULTS.md`, with pass/fail evidence. NO mass traffic tests on production.

## 7. Outputs expected for manager and implementation team

1. Claudally vs MVP comparison matrix and decision (reuse/refactor/clean-room) with licensing implications.
2. High-level architecture plus network sequence/data-flow diagram.
3. Security checklist with identity, auth, authorization, WAF absence/trade-offs and sensitive data exposure.
4. Revised Windows Server 2022 + Caddy runbook with exact commands only AFTER current application code and target infrastructure are confirmed.
5. Copilot Studio migration/configuration guide, dynamic OAuth/client-registration testing and clear note on per-user auth limitations if any.
6. Explicit prerequisites from IT: operating server details, Tally server mode/company, network restrictions, existing static public IP or request, domain+DNS management, TLS issuance, Entra app registration, Power Platform environment access, licensing, change window, approvers.
7. End-to-end test results for company sandbox, then approval-gated production pilot, including rollback/operations.
8. Files `AGENTS.md` and a durable docs folder that enables future developers to understand the product and deployment.

## 8. References

- Claudally repository: https://github.com/JINA-CODE-SYSTEMS/tally-mcp-server
- Claudally installer behavior: https://github.com/JINA-CODE-SYSTEMS/tally-mcp-server/blob/main/docs/installer.md
- Legacy Claudally Cloudflare notes: https://github.com/JINA-CODE-SYSTEMS/tally-mcp-server/blob/main/docs/cloudflare-tunnel-provisioning.md
- Claudally licensing notice: https://github.com/JINA-CODE-SYSTEMS/tally-mcp-server/blob/main/NOTICE
- Microsoft Copilot Studio MCP: https://learn.microsoft.com/en-us/microsoft-copilot-studio/mcp-add-existing-server-to-agent
- Caddy reverse proxy: https://caddyserver.com/docs/caddyfile/directives/reverse_proxy
- OpenAI Codex repo instructions: https://developers.openai.com/codex/guides/agents-md

## 9. Unknowns that Codex MUST verify; do not invent

- Actual location/commit of user's internal MVP, current implementation of its middleware and current Copilot agent connection.
- Access to the company's production Tally VM and whether that is the same VM as any customer's referenced 4 vCPU/14GB machine.
- Current company production Tally version, loaded company(s), approved test data and service behavior outside RDP.
- Azure VM public/private IP, NSGs, VNet topology, corporate inbound policy, hostname ownership, TLS acquisition and approval.
- Actual Claudally HTTP endpoints, OAuth compatibility, CORS/origin behavior, platform binaries and Windows service startup.
- User attribution end-to-end under Copilot's chosen auth flow, and finance data-handling policy.

End of handoff.
