# Proposed decisions

Status: updated after the user selected the Claudally foundation; not deployment authorization. [MODULAR_CLAUDALLY_PLAN](MODULAR_CLAUDALLY_PLAN.md) is the current delivery plan.

| Decision | Rationale / consequence |
|---|---|
| D01: Modular Claudally foundation | One application in this repository; administrator-controlled client profiles. Add review/reporting internally, preserving useful MVP behavior without a second backend. Supersedes the original MVP-foundation recommendation. |
| D02: Preserve license and provenance | Existing AGPL/NOTICE remain; settle compliance/commercial terms for client release. Confirm ownership and licensing before porting MVP implementation. Architecture selection is not legal sign-off. |
| D03: Read-only by construction | Register only approved tools and fixed templates. No raw XML/SQL, GUI, financial writes or company restarts. |
| D04: Native Windows service + Caddy on existing VM | Matches selected cost/hosting constraints; Windows packaging, resource contention and lack of HA/WAF remain acceptance gates. |
| D05: `/mcp` Streamable HTTP only at public boundary | Both codebases support it. Legacy SSE aliases need not be exposed; Streamable responses may still legitimately use text/event-stream. |
| D06: Entra/manual OAuth candidate | Prefer established identity provider and backend token validation. Copilot per-user behavior must be proven; shared key does not establish employee identity. |
| D07: Explicit company per request | Avoid global selection and cross-user races. Preload approved company operationally; no service-side automatic loading. |
| D08: Retain deterministic review with limitations | Preserve existing rules/reports; add versioned finance-approved COA and uncertainty handling. LLM must not decide or apply corrections. |
| D09: Subject-bound reports/sessions | Random tokens and session UUIDs alone do not authorize access. Browser download UX needs staging proof. |
| D10: No production activity during Phase 0 | Documentation only; local implementation is a subsequent phase, followed by explicit staging/production gates. |

Alternatives: adapting Claudally is reasonable only if broad reports are essential and legal/security costs are accepted; building everything from scratch discards useful MVP domain code; Application Gateway/APIM adds capability/cost outside the chosen pilot; temporary tunnels are not production ingress. An outbound/private path must be approved and proven compatible if public ingress is forbidden.
