# Proposed network matrix

UNVERIFIED target policy; no rules changed. Validate actual addresses, IPv4/IPv6 and existing services with IT.

| Source | Destination | Protocol / port | Policy |
|---|---|---|---|
| Approved Copilot service traffic / authorized users where needed | Company hostname -> approved VM ingress | HTTPS TCP 443 | Permit only after approved ingress plan; authenticate in app. Confirm any source restrictions against actual Microsoft environment routing. |
| Internet | MCP/Tally | TCP 3000 / 9000 | Deny; enforce host/NSG and application bind boundaries |
| Caddy on VM | MCP service | HTTP 127.0.0.1:3000 | Permit loopback only |
| MCP service | TallyPrime | HTTP 127.0.0.1:9000 | Fixed destination; XML payloads never traverse public network |
| Browser/Copilot | Entra authorization/token endpoints | HTTPS 443 | Approved identity flow; exact redirect registration |
| MCP service | Entra discovery/JWKS | HTTPS 443 | Allow approved endpoints; validate issuer and key rotation; no arbitrary caller URL fetch |
| Caddy | Certificate issuer / approved DNS API | HTTPS 443, DNS resolution as required | Depends on issuance method; credential and renewal controls |
| Certificate validation authority | Host port 80 or 443 | Challenge-dependent | Open only if selected method requires and IT approves |
| Administrators | Management access | Approved private/Bastion/VPN path | No public RDP exposure |
| Local monitoring | Minimal readiness/metrics | Local/private only | No company payloads; bounded probes |

Report downloads may originate from the employee's browser rather than Copilot cloud. Design authentication and ingress policy for that separate path. Do not hardcode unverified Copilot egress ranges or assume CORS is an access-control substitute for server-to-server traffic. Keep Caddy's administrative listener private and do not add it to public routing.
