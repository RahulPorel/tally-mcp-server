# Tally integration working instructions

Read [PROJECT_HANDOFF.md](PROJECT_HANDOFF.md) and [the Phase 0 analysis](docs/architecture/REPO_COMPARISON.md) before changing code. The handoff is planning context; distinguish source evidence, user-reported outcomes, and unverified deployment assumptions.

- Claudally source is this repository; internal MVP is `D:/Projects/tally-integration-api`. Do not assume folders named `claudally-upstream` or `internal-mvp` exist. Preserve existing work in both repositories.
- Current scope is analysis/documentation. Subsequent local implementation must preserve the MVP's deterministic review/reporting behavior and resolve the licensing decision before copying Claudally code.
- Updated user direction: use this Claudally repository as the sole application codebase, with administrator-controlled modules and a review-only client profile. Follow [MODULAR_CLAUDALLY_PLAN.md](docs/architecture/MODULAR_CLAUDALLY_PLAN.md); it supersedes the initial MVP-foundation recommendation. Preserve license/provenance requirements and do not run a second MVP backend.
- Never access production Tally, install services, publish endpoints, create service accounts, change Azure/Windows networking, DNS, certificates or Tally settings without explicit approval and a documented change window.
- No financial writes, raw model-authored XML, arbitrary SQL, GUI automation or automatic Tally restarts. Even read-only production tests need approval. Use synthetic fixtures first and an approved sanitized company later.
- Target: Copilot Studio Streamable HTTP `/mcp` -> Caddy HTTPS -> native Windows service on `127.0.0.1:3000` -> Tally XML on `127.0.0.1:9000`. Caddy is not an OAuth server or WAF.
- Enforce identity/company/tool authorization in the backend, not prompts. Do not equate connection identity with the employee chatting to the agent.
- Never print or commit secrets or financial payloads. Do not load real `.env` during tests; isolate configuration and block real Tally/GUI access.
- Record decisions, risks and checks in `docs/`. Label unexecuted checks UNVERIFIED. Production readiness requires actual Copilot invocation, authorization negatives, reconciliation, restart/unattended tests and IT/finance sign-off.
- Preserve AGPL and NOTICE attribution. Do not infer MIT licensing from historical upstream provenance.
