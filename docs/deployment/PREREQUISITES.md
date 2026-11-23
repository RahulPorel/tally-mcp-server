# Windows Server 2022 production prerequisites

Planning checklist only. No installation, account creation, production access, DNS, firewall, certificate or Azure changes are authorized by this document. Exact commands require confirmed infrastructure and a separate approved change window.

## Required approvals and inputs

| Area | Required before deployment | Owner |
|---|---|---|
| Change control | Named technical/security/finance approvers; ticket, window, on-call contact, read-only scope, stop criteria and rollback authority | IT / finance |
| Host | Actual Windows Server 2022 build/patch level, CPU/RAM/free disk, existing listeners/services and resource baseline; approved installation/state/log paths | Windows admin |
| Recovery | Current Tally/application configuration backups, documented restore test and appropriate consistent data backup; VM snapshot alone is not financial recovery proof | Tally/backup admin |
| Tally | Version/build/license, approved company loaded, sanctioned XML account/settings, bind/firewall verification, sanitized test company, availability without active RDP | Tally admin |
| Public ingress | Written permission for direct ingress; approved static IP/DNS authority; NSG and Windows Firewall change plan; existing 443 conflicts resolved | Azure/network owner |
| TLS | Company-owned hostname, trusted certificate chain, ACME or enterprise issuance method, storage ACLs, renewal/restart monitoring and expiry alert | PKI/IT |
| Identity | Entra tenant/app registrations, API audience/scopes, exact Copilot-generated redirect URI, consent policy, client credential storage/rotation, JWKS access | Identity admin |
| Power Platform | Environment/agent owner and maker rights, licensing/capacity, DLP policy, connectors allowed, connection references and managed solution strategy | Platform admin |
| Service | Approved pinned Node LTS and WinSW/NSSM versions; dedicated least-privilege accounts, logon-as-service, explicit ACLs and recovery settings | Windows admin |
| Security | Company/tool policy, session/report authorization, limits, dependency scan, log redaction and data-retention policy; no outstanding P0 findings | Security / engineering |
| Finance | Rules/COA version, golden cases, reconciliation method and acceptable exception handling | Finance |
| Operations | Health/readiness distinction, alerts, certificate/token expiry handling, release/rollback manifest and incident ownership | Operations |

No secrets should appear in Git, command transcripts, AI logs or report URLs. Prefer externally managed identity; protect unavoidable service configuration using approved ACLs/DPAPI/secret store under the actual service account. Keep install binaries administrator-owned and non-writable by the service; grant only required state/log rights. Avoid LocalSystem and GUI desktop rights. A service that reads XML does not need blanket access to Tally's financial data directory.

## Staged deployment sequence after approval

1. Finish the local fake-Tally tests and legal/reuse decision. Produce a release with commit, lockfile hash, runtime/wrapper/Caddy versions, dependency inventory and artifact checksums. Rebuild on clean Windows; verify no dependency on a developer profile or working directory.
2. On approved staging only, provision explicit accounts/directories/ACLs and install MCP with absolute executable/config paths. Bind loopback. Set production auth/policy/budgets; fail startup if invalid. Verify recovery and bounded service stop.
3. Validate the already-loaded sanitized company without modifying its records or Tally settings. Test unavailable/closed-company failure and logoff/reboot behavior. If unavailable unattended, stop and resolve with the Tally administrator; do not automate UI/restart as a workaround.
4. Validate Caddy configuration locally with exact path/method allowlists, public Host expectations, auth/session header forwarding, streaming and controlled deadlines. Keep Caddy admin API private. Do not blanket proxy root, legacy aliases, OpenAPI or status.
5. Only under approved network change: apply hostname/IP/NSG/Firewall/TLS configuration. Verify outside access to 443 and denial of direct 3000/9000/public RDP, including IPv6. Verify certificate renewal using the selected challenge method.
6. Create a separate staged Copilot connection, validate identity, discovery and actual tools/call results, then finance reconciliation and report retrieval. Preserve the old connection until sign-off.
7. Production pilot uses the approved artifact and bounded read-only scope, with monitoring and rollback owner present. Record results; do not convert one successful call into a general production-readiness claim.

## Caddy routing contract

Target routes: POST/GET/DELETE `/mcp` as required by chosen Streamable mode; required protected-resource metadata; separately approved report retrieval route. OPTIONS only where needed. Reject other routes by default. Proposed Entra authorization/token endpoints live at the IdP, not Caddy. Do not copy Claudally `/authorize`, `/token`, `/register` unless explicitly choosing its authorization server.

If evaluating Claudally unchanged in isolated staging, its discovered paths are both protected-resource metadata routes, authorization-server metadata, `/authorize` GET/POST, `/token`, `/register`, and `/mcp`. Test registration and callbacks deliberately; do not publish `/status` or every app route.

Preserve external Host where backend Host validation requires it; trust forwarded headers only from the known loopback proxy. Confirm `Authorization`, `Mcp-Session-Id`, `MCP-Protocol-Version`, Accept and response content types survive. Streamable HTTP can use event-stream responses; this does not mean the old `/sse` transport is required. Set stream/deadline behavior from tested client limits, and avoid automatic retries that repeat expensive Tally work.

Caddy is not a WAF or OAuth enforcement service in this design. Request body controls and application quotas remain mandatory. ACME HTTP-01 may require 80; TLS-ALPN-01 uses 443; DNS-based issuance needs controlled DNS credentials/provider support. Select and approve one method rather than opening 80 by habit. Ports, DNS and certificate issuance are all change-controlled.

## Cost and compatibility limits

No new VM, Docker Desktop, Application Gateway WAF or APIM is assumed. Existing host resource contention, a single failure domain, public IP/DNS/certificate operations, Copilot licensing/capacity and monitoring still have costs. A Power Platform on-premises data gateway is not assumed to transparently support this MCP flow. If inbound policy disallows direct access, mark this deployment path BLOCKED and design an approved alternative before implementation.
