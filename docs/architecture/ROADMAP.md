# Implementation roadmap

**Superseded implementation sequence:** the user selected modular Claudally as the sole application codebase. Follow the ordered steps in [MODULAR_CLAUDALLY_PLAN](MODULAR_CLAUDALLY_PLAN.md). The earlier MVP-based sequence below remains historical planning context; staging/production approval and validation requirements still apply.

Phase 0 delivered source analysis, comparison, architecture, security review and deployment/test gates. No production readiness is claimed. Execution evidence remains in [VALIDATION_PLAN](../deployment/VALIDATION_PLAN.md).

| Phase | Work / owner | Exit evidence |
|---|---|---|
| 1A: Preserve baseline | Engineering: capture MVP release/connection configuration without credentials; isolate fixtures and configuration; add complete MCP initialize/list/call test harness using fake Tally only | Original four tools and workbook behavior captured; no test can reach real Tally/GUI; reproducible locked build |
| 1B: Secure boundary | Engineering/security: fail-closed auth, Entra token validator, shared schemas, principal/company/tool policy, session ownership/expiry, explicit error handling, remove public legacy routes and URL-key auth | Missing/expired/wrong-tenant/audience/scope credentials rejected; cross-company/session negatives pass before adapter calls |
| 1C: Safe finance core | Engineering/finance: strict amounts/dates/response envelopes, company-qualified IDs, bounded query queue and result sizes, rule/COA versioning, ambiguity handling, private reports | Synthetic reconciliation fixtures; malformed values never become success; signed totals preserved; rule golden cases and report owner/expiry tests pass |
| 1D: Minimal capability extension | Engineering: fixed company-list and one approved ledger/report template; independently implement or legally clear reuse | Correct escaped template, no caller-authored XML/SQL; fixture error/encoding coverage; authorized results only |
| 2: Windows staging | IT/engineering: approved sanitized host/company; pinned service wrapper and Node release; least-privilege accounts; local Caddy trial; auth/proxy contracts; shutdown/restart tests | Service recovery, logoff/reboot and loaded-company availability; ACLs and local binds verified; resource budgets met |
| 3: Copilot staging | Power Platform owner/security/finance: separate connection using stable staging HTTPS; manual OAuth; employee and shared-connection attribution tests; report browser UX | Actual tools/list and calls in agent/channel; two-user denials; token refresh; Tally error behavior; reconciled sandbox report; no secrets in transcripts/logs |
| 4: Restricted production pilot | Named IT/security/finance approvers: change ticket/window, backup/recovery evidence, approved ingress/DNS/TLS, release and rollback; narrow read-only data/date scope | Low-volume approved UAT; financial reconciliation; monitoring and rollback rehearsal; written sign-off |
| 5: Client rollout | Product/legal/operations: managed Power Platform solution, client-owned identity/credentials/config, compatibility checklist and runbooks | Each client independently passes network, Tally/license, tenant, isolation and service availability gates |

Do not promise dates until identity, VM policy and Tally operational constraints are answered. Prioritize P0 authorization/report exposure and correctness before adding broad reports. Keep old MVP connection available during staged migration; do not switch the live agent as part of local code work.

Completion requires documentation and evidence alongside code: versioned release manifest/SBOM, configuration schema, service/proxy configuration validated for the actual host, threat-model updates, UAT results, approved network matrix, monitoring ownership and rollback. Exact production commands are intentionally deferred until installation paths, service identities, certificate method, hostname and change approvals are known.
