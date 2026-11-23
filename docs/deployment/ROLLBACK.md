# Rollback and incident plan

Proposed procedure; execution requires the approved change ticket and named operator. No Tally restart, data restore or Azure/DNS mutation is implicitly authorized here.

## Prepare before pilot

- Retain the previous approved application artifact, dependency/runtime manifest, service/proxy configurations and ACL/account definitions. Keep secrets in the approved store, not release archives or Git.
- Record the working Copilot connection and solution version; stage the replacement separately. Capture existing ingress/DNS/certificate configuration with owners and restoration steps.
- Verify backups and Tally-specific recovery with its administrator. Agree RTO/RPO, rollback triggers and who can stop the pilot.

## Trigger and containment

Trigger on incorrect-company access, unexpected mutation, reconciliation mismatch, credential compromise, certificate failure, service crash loop or Tally/resource disruption. Stop new integration requests using the approved app/service or ingress control; preserve minimal redacted evidence. Disable the new Copilot connection if authorized. Do not kill arbitrary Node processes or stop Tally to recover the integration.

## Restore

1. Stop only the identified integration service under the approved procedure and drain/cancel its work within a deadline.
2. Restore the known-good application/configuration and compatible runtime/service definition; validate configuration and local health. If the old release has a security defect, keep the integration disabled instead of restoring unsafe exposure.
3. Restore only ingress/DNS/Caddy changes listed in the ticket, with their responsible admin. Account for DNS TTL and certificate storage; do not remove shared VM networking or certificates used by other services.
4. Re-enable the old Copilot connection only if it remains approved and safe. Reinitialize MCP sessions; expect in-memory reports/session IDs to be lost. Notify affected users to regenerate reports through the authorized flow.
5. Rotate/revoke credentials if compromised and enforce reauthentication. Do not blindly restore old token stores that would revive revoked sessions.
6. Verify minimal approved read-only reconciliation and service availability. Finance owns any decision to restore financial data if unexpected changes occurred; this read-only integration rollout should require no Tally data migration.

Record incident timing, version, scope, redacted evidence and approval. Keep the pilot paused until cause and corrective tests are signed off. A successful rollback is a tested operational procedure, not merely the existence of a backup ZIP.
