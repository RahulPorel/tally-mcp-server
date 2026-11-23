# Phase 0: TallyPrime integration comparison

Assessment date: 2026-11-23. Scope: local source and configuration review, test inspection, and Microsoft/Caddy documentation. No application server, installer, financial operation, production probe or network change was executed. Existing `.env` contents were not inspected. Tests were inspected, not run; this report is not runtime certification.

## Recommendation

**Direction updated by the user:** use Claudally as the sole application codebase with administrator-configured modules and a dedicated review profile. See [MODULAR_CLAUDALLY_PLAN](MODULAR_CLAUDALLY_PLAN.md) for the current plan. The recommendation below is retained as the original Phase 0 assessment; its MVP-foundation choice is superseded, while the source/security findings remain applicable.

Use the internal MVP as the domain/application foundation. Preserve its four MCP tools, deterministic review engine, normalization contract and Excel reports, then harden the transport, identity, company authorization, financial validation and Windows operations. Treat Claudally as a comparative reference for report coverage, failure cases and test scenarios. Do not merge or copy its implementation pending a legal decision.

Claudally is substantially broader than the requested read-only review pilot. Its company loading, GUI, credential vault, arbitrary SQL and posting machinery add both operational dependencies and security scope. Its existing read-only switch does not remove company restart/switch capabilities. Adapting the whole product would mean removing a large part of it while still building employee authorization and domain review logic.

## Repository identity and provenance

| Label used in this report | Actual checkout | Local HEAD | State before analysis |
|---|---|---|---|
| claudally-upstream | `D:/Projects/tally-mcp-server` | `74869e5388da3ba5c30fbd01ef72af4ca32d38fd` | Clean; origin JINA-CODE-SYSTEMS/tally-mcp-server; version 0.10.0 |
| internal-mvp | `D:/Projects/tally-integration-api` | `29818c0b5525b906606da5063045db0236b9b5df` | Two pre-existing untracked documents: ARCHITECTURE.md and ENTERPRISE_DEPLOYMENT_GUIDE.md; version 1.0.0 |

These are descriptive labels, not discovered subdirectories. Remote HEAD freshness was not checked; these SHAs pin the code actually reviewed. Existing generated output is not proof of a reproducible release. The user's successful Copilot Studio use is accepted as user-reported evidence; the connected endpoint, credentials, tenant settings and invocation trace have not been independently inspected.

## Feature and gap matrix

| Capability | Claudally source | Internal MVP source | Pilot decision |
|---|---|---|---|
| MCP transport | stdio entrypoint; HTTP POST/GET/DELETE `/mcp` | Stateful Streamable HTTP plus legacy SSE multiplexed over aliases | Retain Streamable HTTP; remove legacy ambiguity from public routing |
| Tools | 55 direct registrations in mcp.mts plus feeding-guide helper | list_tally_companies, review_vouchers, search_day_book, check_api_health | Keep narrow explicit allowlist |
| Financial reads | Masters/COA, trial balance, P&L, balance sheet, stock, bills, ledger, party and GST reports | Company list and normalized Day Book vouchers | Preserve MVP; add one approved ledger/report template later |
| Financial writes | Create/batch/alter/reverse/delete vouchers; masters, GST and party edits | No exposed financial write tools found | Exclude all writes from pilot binary/tool registration |
| Company operations | Disk discovery, aliases, active company, load/restart/switch, GUI and stored credentials | Explicit company parameter per query; no loading | Require preloaded approved company; fail closed if unavailable |
| Review | No equivalent five-rule review/Excel pipeline identified | Five hardcoded expense categories, narrative period/type observations, line-level evidence | Reuse with finance-approved limitations and rule versioning |
| Excel | No equivalent MVP workflow identified | Summary, Findings and Vouchers workbook; text sanitization; in-memory download cache | Reuse generator; replace download authorization/storage policy |
| Authentication | Shared password authorizes OAuth codes; PKCE S256, DCR, bearer/refresh tokens | Disabled by default or one shared key; header/Bearer/query accepted | Entra-backed OAuth resource validation; fail startup on insecure production config |
| Authorization | No employee/company policy; global active company and shared DuckDB | No employee/company policy; shared sessions/report cache | New principal/company/tool policy layer |
| SQL/cache | In-memory DuckDB; SELECT filtering, external access disabled; 15-minute tables | No database; one-hour report cache | No arbitrary SQL; bounded, owner-scoped report storage |
| Windows packaging | Local stdio installer; remote NSSM script; interactive GUI companion | Node build/start; Linux Docker sample; no native service installer found | New minimal Windows release/service packaging, Caddy |
| Tests | 60 test files; auth, transport, XML, company, writes, read-only regression suites | 8 test files: auth, MCP, XML/client, normalization, review, workbook, REST | Extend fixture suite; prove end-to-end Streamable HTTP and isolation |
| License | AGPL-3.0-or-later plus NOTICE attribution | package declares ISC; ownership/provenance still needs confirmation | Internal-base approach; legal gate before upstream copying |

## Dependency map and exact lockfile samples

| Dependency | Claudally lock | MVP lock | Relevance |
|---|---|---|---|
| MCP SDK | 1.32.2 | 1.32.2 | Same transport library; migration need not replace business logic |
| Express | 5.3.0 | 5.3.0 | HTTP boundary |
| fast-xml-parser | 5.12.1 | 5.11.2 | XML compatibility and parser hardening |
| TypeScript | 7.0.2 | 7.0.2 | Build toolchain |
| Zod | 4.6.7 (resolved) | 4.6.7 | Validation; Claudally imports it without listing it directly in its manifest |
| DuckDB node API | 1.4.1-r.4 | Absent | Native binary/platform verification and shared cache risk |
| ExcelJS | Absent | 4.4.0 | Existing workbook reuse |
| Vitest | Absent; node:test | 5.0.5 | MVP fixtures |

Claudally declares Node >=22 and CI targets Node 22/24 on Linux. MVP has no engines field; its Docker example uses Node 22 Alpine. These are not Windows Server 2022 compatibility results. Manifests use ranges; preserve lockfiles and release hashes, choose a supported Node LTS patch, run clean Windows builds and dependency/license scans before packaging. No vulnerability scan result is claimed here.

## Source evidence and reuse boundaries

- Claudally [HTTP app](../../src/http-app.mts), [entrypoint](../../src/server.mts), [stdio entrypoint](../../src/index.mts), [OAuth store](../../src/oauth-store.mts), [tool registry](../../src/mcp.mts), [Tally client](../../src/tally.mts), [DuckDB](../../src/database.mts), [feeding guide](../../src/feeding-guide.mts), [NOTICE](../../NOTICE), [installer](../installer.md), [service script](../../scripts/setup-windows.ps1), [CI](../../.github/workflows/ci.yml).
- MVP files under `D:/Projects/tally-integration-api`: src/app.ts; src/server.ts; src/config/env.ts; src/mcp/server.ts; src/middleware/auth.ts; src/routes/{health,companies,vouchers,reviews,reports}.ts; src/schemas/voucher.ts; src/services/{tally-client,tally-xml,voucher-normalizer,review-engine,report-generator,report-cache}.ts; src/types; tests; openapi/copilot-openapi.json; package and lockfile; Docker files; existing architecture/deployment/handoff material.
- Reuse the MVP review engine and workbook generator after regression tests, rather than replacing deterministic processing with LLM decisions. Its rules correspond to the supplied AccountingRules.txt excerpt. No runtime DOCX/XLSX rule/COA ingestion was found; the actual reference workbook content was not validated in this phase.
- Refactor its XML boundary, normalization error policy, report cache and auth/session handling. Use a shared validation contract for REST and MCP.
- New work: OAuth validation, policy enforcement, subject-bound sessions/downloads, bounded workload controls, minimal audit metadata, readiness, service packaging, proxy configuration, release/rollback process.
- If broad reporting later warrants Claudally reuse, legal must decide AGPL compliance/source availability and attribution obligations or negotiate commercial terms. Merely moving copied code behind HTTP does not settle derivative-work questions. This review has already examined upstream source; do not label later work a legally certified clean-room implementation.

## Documentation corrections

Claudally's older security assessment is historical: current source requires PASSWORD, validates all three MCP verbs, checks client credentials, rotates refresh tokens, restricts CORS/Host, and blocks DuckDB external access. Remaining risks are listed separately in [SECURITY_REVIEW](../security/SECURITY_REVIEW.md); old fixed issues are not repeated as current findings.

The current Windows installer is local-only for new installations. Remote setup exists separately; neither is a turnkey Caddy/Entra/Copilot enterprise installer. Legacy Cloudflare instructions are not the target deployment.

The MVP's existing documents overstate guarantees: memory caches persist beyond a request, a 100 KB request-body limit does not bound Tally responses or Excel memory, deterministic matching can still misclassify entries, and Export alone is not a universal proof that arbitrary TDL has no side effects. Application Gateway/APIM designs in those documents are alternatives, not evidence of deployed infrastructure.

The [TDL experiment notes](../../notes/tdl-experiments.md) are historical reports, not reproduced results. They describe modal-blocking probes and restart/GUI loading. Their universal claims about XML loading and Silver/Gold company limits require vendor confirmation; do not base production licensing or design on those claims alone. No further probes are authorized by this analysis.

## Deliverable index

- [AS_IS](AS_IS.md): component and dependency flows.
- [TO_BE](TO_BE.md): target trust boundaries and request sequence.
- [SECURITY_REVIEW](../security/SECURITY_REVIEW.md): prioritized findings and acceptance conditions.
- [DECISIONS](DECISIONS.md), [ROADMAP](ROADMAP.md), [OPEN_QUESTIONS](OPEN_QUESTIONS.md).
- [PREREQUISITES](../deployment/PREREQUISITES.md), [NETWORK_MATRIX](../deployment/NETWORK_MATRIX.md), [COPILOT_MIGRATION](../deployment/COPILOT_MIGRATION.md), [ROLLBACK](../deployment/ROLLBACK.md), [VALIDATION_PLAN](../deployment/VALIDATION_PLAN.md).
