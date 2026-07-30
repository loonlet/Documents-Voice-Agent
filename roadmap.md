# Roadmap and Implementation Specifications

This roadmap is derived from confirmed gaps in `docs/features.md`. Items below are framed as implementation-ready specs with acceptance criteria.

## Phase 0: Build the Runtime Baseline (Critical)

### R0.1 Backend service foundation (Not Implemented)

- Build Python backend services for API, orchestrator, and workers.
- Add dependency manifests, environment templates, and configuration loading.

Acceptance criteria:

- `api`, `orchestrator`, and `worker` services start locally via Docker Compose.
- Health/readiness endpoints return healthy states.
- Structured logs and trace IDs present in each service.

### R0.2 Telephony runtime path (Partial -> Implemented)

- Complete FreeSWITCH install/config, SIP trunk, dialplan, and audio streaming.

Acceptance criteria:

- Inbound and outbound calls complete successfully in staging.
- AI route and human fallback route both functional.
- Call event stream reaches backend APIs.

## Phase 1: Product Gaps from Prompt (Required)

### R1.1 UI/frontend: lead and call dashboard (Not Implemented)

Scope:

- Lead funnel view, callback SLA tracker, call outcome board.
- Filter by status, language, campaign, and owner.

Acceptance criteria:

- Dashboard updates near real-time (<5s lag) **[ASSUMPTION]**.
- SLA breaches visibly highlighted and alertable.
- Role-based access applied.

### R1.2 UI/frontend: agent configuration screen (Not Implemented)

Scope:

- Configure prompts, escalation rules, language defaults, recording policy, model selection.

Acceptance criteria:

- Versioned config changes with audit trail.
- Safe rollback to previous config version.
- Validation prevents invalid runtime combinations.

### R1.3 UI/frontend: live call monitoring screen (Not Implemented)

Scope:

- Active-call list, transcript stream, sentiment indicator, manual handoff button.

Acceptance criteria:

- Agents can monitor active sessions without page reload.
- Manual handoff reaches telephony route in under 2 seconds **[ASSUMPTION]**.
- All operator actions are audit logged.

### R1.4 CRM bi-directional sync (Not Implemented)

Scope:

- Sync lead/call/appointment records to CRM and back.
- Conflict-resolution policy and id mapping.

Acceptance criteria:

- Create/update events sync both directions with idempotency.
- Sync failures retry automatically and surface in ops dashboard.
- Data model mapping documented and tested.

### R1.5 WhatsApp integration (Not Implemented)

Scope:

- Send confirmations/follow-ups; process inbound replies; enforce opt-in/opt-out.

Acceptance criteria:

- Booking confirmation is sent with delivery state tracking.
- STOP/START keywords change contact consent state immediately.
- Failed deliveries retried with fallback to email/SMS policy.

### R1.6 Email notification service (Not Implemented)

Scope:

- Transactional templates for booking confirmation, callback outcome, escalation alert.

Acceptance criteria:

- Templated notifications support localization.
- Delivery metrics and bounce/complaint handling available.
- PII-safe logging enforced.

### R1.7 RAG knowledge base for FAQ/pricing/policies (Not Implemented)

Scope:

- Ingestion pipeline, chunking, embeddings, vector store, hybrid retrieval.

Acceptance criteria:

- FAQ responses include source metadata.
- Retrieval quality passes offline benchmark set.
- Hallucination guardrails trigger fallback/handoff.

### R1.8 Formal QA program (Not Implemented)

Scope:

- Unit/integration tests, UAT sign-off, regression suite, load/security tests.

Acceptance criteria:

- CI gate enforces minimum coverage threshold **[ASSUMPTION]**.
- UAT sign-off checklist completed before release.
- Regression suite green on every release candidate.

## Phase 2: Advanced OSS Features

### R2.1 Low-latency streaming improvements

- Full duplex streaming STT/TTS, chunked synthesis, and jitter-buffer tuning.

Acceptance criteria:

- p95 turn latency under agreed threshold.
- Stable performance at target concurrency.

### R2.2 Function-calling and tool orchestration

- Explicit tool-use policies for CRM, booking, and notification actions.

Acceptance criteria:

- Tool calls are schema-validated and auditable.
- Unsafe or ambiguous tool calls are rejected.

### R2.3 Intent/sentiment classification and barge-in

- Add classifiers to support dynamic handoff and interruption handling.

Acceptance criteria:

- Escalation precision/recall metrics tracked.
- Barge-in behavior improves interruption UX measurably.

### R2.4 Observability maturity

- OpenTelemetry traces across telephony + AI + integrations.

Acceptance criteria:

- Single-call trace from webhook/call start to resolution available.
- Alerts for callback SLA, error budget, and model latency configured.

## Phase 3: Governance and Compliance

### R3.1 Data rights and retention automation

Acceptance criteria:

- Export/delete workflows operational per tenant.
- Recording retention and purge jobs verifiable by audit logs.

### R3.2 Security validation program

Acceptance criteria:

- Threat model updated quarterly.
- External pen test issues resolved to acceptable risk.

## Suggested Ticket Breakdown

- Epic: `E1 Runtime Core`
- Epic: `E2 Telephony + AI Loop`
- Epic: `E3 Integrations (CRM/WhatsApp/Email/Calendar)`
- Epic: `E4 Dashboard and Operations UX`
- Epic: `E5 QA, Performance, and Security`

For architecture details, see `docs/architecture.md`. For security controls, see `docs/security.md`.

