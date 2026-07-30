# Security and Compliance

Repository evidence indicates security is currently a backlog item (`AIVOICEAGE-14` in `src/AIVOICEAGE_board.html`). This document defines the production security baseline.

## 1) Identity, Authentication, and Authorization

- Dashboard auth: OpenID Connect (self-hosted IdP such as Keycloak) with MFA for all admin users.
- API auth:
  - Machine-to-machine: short-lived JWTs signed by internal issuer.
  - Webhooks: HMAC signatures + nonce + timestamp.
- Authorization:
  - Role-based access control (`admin`, `ops`, `qa`, `sales`, `readonly`).
  - Tenant/account scoping for call and lead data.
- Session management:
  - 15-minute idle timeout for privileged screens **[ASSUMPTION]**.
  - Device-bound refresh tokens with revocation support.

## 2) Encryption in Transit and At Rest

- Transit:
  - TLS 1.2+ for all HTTP APIs.
  - SIP over TLS + SRTP for media wherever provider supports it.
  - mTLS between internal services in Kubernetes **[ASSUMPTION]**.
- At rest:
  - Full-disk encryption on hosts.
  - Database encryption and row-level access controls.
  - Object storage encryption (SSE) for recordings/transcripts.
- Key management:
  - Centralized KMS/Vault.
  - Separate keys for DB, object storage, webhook signing, and JWT.

## 3) SIP/Telephony Hardening (Fraud and Toll-Abuse Prevention)

- Restrict SIP signaling to trusted peer IP ranges.
- Use fail2ban or equivalent for brute-force protection.
- Disable anonymous/guest calls on FreeSWITCH.
- Enforce strong SIP credentials and regular rotation.
- Apply dial-permission policy (country/number prefix allowlists).
- Configure per-trunk rate limits and CPS (calls per second) caps.
- Block premium/international routes by default unless explicitly approved.
- Alert on anomaly patterns: burst dialing, repeated failed auth, after-hours spikes.

## 4) Secrets Management

- Do not store API keys in source control.
- Use Vault or encrypted secrets (SOPS + age) for all environments.
- Rotate secrets automatically where supported.
- Record secret-access audit logs.
- Enforce separate credentials for dev/stage/prod.

## 5) PII and Call Data Handling

- Data classes:
  - P0: auth material, tokens, signing keys.
  - P1: phone numbers, email, call audio, transcripts.
  - P2: aggregated metrics and anonymized analytics.
- Minimization:
  - Persist only required lead and support fields.
  - Redact payment details from transcripts.
- Access controls:
  - Principle of least privilege per service and user role.
  - Break-glass access with explicit approval logging.

## 6) Data Retention and Deletion Policy

- Call recordings retention: 30-180 days by policy/region **[ASSUMPTION]**.
- Transcript retention: configurable per tenant.
- Lead/contact retention: tied to business purpose and legal basis.
- Deletion support:
  - Soft delete for operational rollback window.
  - Hard delete workflow for regulatory requests.
- Backup retention and secure destruction policy required.

## 7) Compliance Considerations for Voice Platforms

- Recording consent:
  - Pre-call or early-call consent prompt.
  - Jurisdiction-aware one-party/two-party consent logic.
- GDPR-style controls:
  - Right of access/export.
  - Right to rectification.
  - Right to erasure.
  - Right to restrict processing.
  - Data portability.
- Auditability:
  - Immutable event logs for call decisions and handoffs.
  - Traceability from lead capture -> call -> outcome -> notifications.

## 8) Security Monitoring and Incident Response

- Centralize logs in Loki/ELK with retention and tamper controls.
- Alerting coverage:
  - auth failures
  - webhook signature failures
  - toll-fraud anomalies
  - PII access anomalies
  - service downtime and SLA breach
- Incident response runbooks:
  - SIP compromise
  - webhook replay attack
  - data leakage
  - model prompt-injection abuse

## 9) Application Security Controls

- Input validation on all webhook/API payloads.
- Idempotency keys for lead capture and outbound call jobs.
- Rate limiting and circuit breakers for integration endpoints.
- Prompt-injection resistance:
  - isolate tools behind explicit allowlists
  - strip unsafe tool instructions from user text
- Dependency scanning and container image scanning in CI.

## 10) Security Acceptance Checklist (Go-Live Gate)

- [ ] OIDC + MFA enabled for dashboard and admin APIs
- [ ] SIP TLS/SRTP enabled and verified end-to-end
- [ ] Secrets externalized and rotated
- [ ] Data retention and deletion workflows tested
- [ ] Recording consent flows validated for target jurisdictions
- [ ] SIEM/alerts connected and on-call ownership assigned
- [ ] Pen test and remediation completed

For API-level controls, see `docs/api-reference.md`. For deployment controls, see `docs/setup.md`.

