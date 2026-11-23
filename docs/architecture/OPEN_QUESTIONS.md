# Open questions and evidence requests

All deployment details below are UNVERIFIED. These are implementation prerequisites, not requests to send secrets or upload confidential accounting data.

| Question | Owner / minimum evidence | Blocks |
|---|---|---|
| Which connection currently works in Copilot: MCP wizard or REST connector, URL path, auth mode, agent/channel/environment? | Power Platform owner: redacted configuration and correlation/timestamp of a successful tool call | Safe migration and attribution baseline |
| Is the connection employee-owned or shared by the agent? | Tenant owner: two-user invocation evidence and validated backend claims | Employee-scoped policy |
| Does policy permit internet ingress to this existing VM? Is 443 already used? | Azure/IT: approved topology, ingress policy, listener inventory | Direct Caddy path; BLOCKED if prohibited |
| Actual VM OS/build, CPU/RAM/disk, backup and maintenance owners? | IT: inventory and restore evidence; do not assume sample 4 vCPU/14 GB/150 GB | Sizing and same-host risk |
| Tally version/build/license, XML bind/security, concurrent users, company load behavior after disconnect/logoff/reboot? | Tally admin/vendor plus approved sanitized tests | Unattended service viability |
| Approved sandbox/company IDs/date ranges and permitted employee roles? | Finance/security: explicit mapping and data classification | Every query and UAT |
| Can Tally's XML interface enforce a sufficiently restricted user on this build? | Tally admin: documented behavior and negative tests | Defense in depth; backend policy still mandatory |
| Domain/static public IP/DNS authority, IPv6 exposure and certificate method? | IT: approved resources and renewal owner | TLS/public routing |
| Entra app registration, tenant restrictions, delegated scopes, consent and callback ownership? | Identity/Power Platform admins | OAuth proof of concept |
| What data may enter cloud transcripts, logs and downloadable reports, and for how long? | Security/legal/finance | Retention and report delivery |
| Is internal MVP wholly company-owned; is AGPL acceptable or a commercial license desired? | Legal/product | Any Claudally code/template copying |
| Which COA version and accounting rules are authoritative; how should tax/multi-expense/foreign currency/optional/cancelled vouchers be treated? | Finance: sanitized golden cases | Correctness acceptance |
| What latency/throughput, RTO/RPO and support coverage are required? | IT/finance | Budgets, service recovery and rollback |

The supplied TDL notes do not settle Silver/Gold multi-company constraints or prove universal absence of an XML loading facility. Get vendor confirmation; the pilot avoids automatic loading either way.
