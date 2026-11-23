# Acceptance tests and evidence ledger

Status on 2026-11-23: source/configuration/test inspection completed. No build, test suite, service start, live MCP call, Copilot tenant operation or Tally probe executed in this Phase 0 analysis. Static review is intentionally separate from runtime pass/fail evidence. Existing test files are not passing test results.

Documentation checks: all 13 new instruction/report files containing local Markdown links were checked; zero broken local links. The local PROJECT_HANDOFF.md copy has the same SHA-256 as the supplied original. A documentation-check attempt found `node` unavailable on this tool shell's PATH; this is a workstation/tool-environment observation, not evidence that Node is absent from the target VM. PowerShell completed the link check. No source/runtime files were changed.

## Existing coverage inspected

Claudally has 60 test files, including HTTP auth/transport, read-only guards, XML/template escaping, company selection and write invariants. HTTP auth harness uses an ephemeral listener and temporary OAuth state, but imports the tool module which loads the install-directory `.env`; therefore do not run wholesale against this configured checkout without isolation. CI runs Linux Node 22/24, not proof of Windows service behavior.

MVP has eight test files: auth, MCP, report generator, review engine, Tally client, XML, normalizer and REST integration. REST tests use loopback ephemeral HTTP and mocked Tally methods; MCP tests inspected cover construction and missing/unknown legacy session IDs, not a complete real Streamable negotiation. Neither repository's current deployment was exercised.

## Required checks

| ID | Test / expected evidence | Stage | Current result |
|---|---|---|---|
| V01 | Clean locked Windows build; pinned runtime; dependency/security/license report; no secrets in artifact | Local isolated build | UNVERIFIED |
| V02 | Full Streamable initialize -> initialized -> tools/list -> tools/call using SDK client/inspector and fake Tally; correct content types/headers | Local | UNVERIFIED |
| V03 | No/malformed/expired/wrong issuer/tenant/audience/scope tokens fail on every verb; auth precedes adapter | Local/staging | UNVERIFIED |
| V04 | Two principals: denied company/tool, guessed session, other user's report, session deletion and refreshed credentials | Local/staging | UNVERIFIED |
| V05 | Only approved tools visible/callable; raw XML/SQL/write/GUI/company-restart attempts rejected without Tally traffic | Local | UNVERIFIED |
| V06 | Valid empty dataset distinguished from no company, STATUS/LINEERROR, bad shape, malformed XML, DTD/entity, truncation and non-2xx | Fake Tally | UNVERIFIED |
| V07 | UTF-8/UTF-16LE, Unicode names, escaping, invalid dates and bounded server-side date filters | Fixtures then sanitized Tally | UNVERIFIED |
| V08 | Invalid/missing money rejected; decimal/sign handling, GUID/company/revision IDs, totals, tax/multi-line/cancelled/optional/FX cases | Finance golden fixtures | UNVERIFIED |
| V09 | Rule version and evidence; ambiguous/multi-rule narration requires review; malicious narration cannot trigger commands | Local/finance | UNVERIFIED |
| V10 | Excel valid, text safe, correct totals/IDs; report expiry/revocation/ownership; no reusable key in links/logs | Local/staging | UNVERIFIED |
| V11 | Byte/row/date/session/cache/queue limits; cancellation and slow/unavailable Tally; no false successful report | Synthetic only | UNVERIFIED |
| V12 | Service start/stop/recovery/reboot/logoff and RDP disconnect; actual least-privilege account and directory ACLs | Approved Windows staging | UNVERIFIED |
| V13 | Caddy Host/Origin/trusted proxy, auth/session headers, streaming, renewal, denied public 3000/9000/RDP incl. IPv6 | Approved staging | UNVERIFIED |
| V14 | Actual Copilot tools invocation and refresh under two users in intended channel; backend identity matches policy | Approved tenant/staging | UNVERIFIED |
| V15 | Finance reconciliation against identical company/date scope, signed totals/counts and representative voucher evidence | Approved sanitized company | UNVERIFIED |
| V16 | Rollback rehearsal and monitoring/alert ownership; no Tally disruption | Approved staging | UNVERIFIED |
| V17 | Approved low-volume production UAT with release/change ticket and named sign-off | Explicit production gate | NOT AUTHORIZED |

Before executing code, create an isolated test configuration without real `.env`, use synthetic XML and deny access to real Tally/GUI/process controls. Do not treat READONLY_MODE as sufficient test isolation. Inspect the test call graph and destination configuration first. Tests that write reports/temp files must stay in an authorized test location.

For each later result record: exact commit/artifact, environment/build, sanitized fixture or approved company/range, test command/client version, timestamp/correlation ID, expected/actual outcome, reviewer and issue link. Keep raw financial payloads outside Git in approved evidence storage. Production UAT must be low-volume, read-only and stop immediately on company mismatch, financial mismatch, unauthorized access or host impact.
