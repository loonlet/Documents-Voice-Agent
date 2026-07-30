# Testing and QA

## Current Coverage (Repository Evidence)

| Area | Status | Evidence |
|---|---|---|
| Automated unit tests | Not Implemented | No test files in repository |
| Integration tests | Not Implemented | No backend services or test harness present |
| UAT sign-off docs | Not Implemented | No UAT artifacts found |
| Regression suite | Not Implemented | `AIVOICEAGE-27` exists only as backlog spec in `src/AIVOICEAGE_board.html` |
| Latency benchmark suite | Not Implemented | `AIVOICEAGE-26` backlog item only |

## Quality Strategy (Target)

## 1) Test Pyramid

- Unit tests: normalization, validators, routing decisions, policy engines.
- Contract tests: CRM, calendar, WhatsApp, email, SIP event payload contracts.
- Integration tests: full STT -> LLM -> TTS turn loop with mocked telephony.
- End-to-end tests: real SIP call in staging, booking and notifications included.
- Non-functional tests: load, soak, failover, security, and data retention tests.

## 2) Required Test Suites by Capability

| Capability | Must-have tests | Pass Criteria |
|---|---|---|
| Lead webhook ingest | schema validation, signature verification, idempotency | Duplicate events do not create duplicate leads |
| Outbound callback SLA | queue timing, retry schedule, after-hours rules | 95% callbacks initiated within SLA window **[ASSUMPTION]** |
| STT/LLM/TTS loop | transcript quality, response relevance, synthesis latency | Median turn latency and quality thresholds met |
| VAD/barge-in | interruption handling, endpointing robustness | Agent avoids cut-offs and over-waits |
| Booking flow | slot lookup, conflict handling, confirmation dispatch | No double-booking in concurrent scenarios |
| Handoff flow | escalation trigger correctness, context transfer integrity | Human agent receives full context payload |
| Recording/compliance | consent gating, retention purge, export/delete rights | Policy and compliance checks pass |

## 3) Suggested Tooling (OSS)

- Python test runner: `pytest`
- API contract and integration: `schemathesis`, `httpx`, `pytest-asyncio`
- Load testing: `k6` or `Locust`
- SIP scenario testing: `sipp`
- Audio quality/regression harness: custom dataset + WER/CER metrics **[ASSUMPTION]**
- CI: GitHub Actions/Gitea CI/Jenkins (self-hosted)

## 4) Test Data and Environments

- Synthetic PII only in CI/staging.
- Gold-call corpus:
  - FAQ calls
  - missing-profile calls
  - purchase-nudge calls
  - booking calls
  - escalation calls
- Multi-language corpus covering top 5 supported locales **[ASSUMPTION]**.

## 5) UAT and Release Gates

- UAT stakeholders: support ops, sales ops, compliance, telephony admin.
- Mandatory sign-offs:
  - Functional completion
  - Security/compliance controls
  - Operational readiness
- Release gate metrics:
  - callback SLA
  - call completion rate
  - escalation precision
  - booking success rate
  - transcription WER by language

## 6) Regression Matrix (Template)

| Test ID | Scenario | Input | Expected | Owner | Status |
|---|---|---|---|---|---|
| REG-001 | FAQ answer | Pricing question | Accurate, concise answer with source grounding | QA | Planned |
| REG-002 | Missing profile field | No email in lead | Agent requests and validates email | QA | Planned |
| REG-003 | Purchase nudge | Abandoned checkout | Agent offers next-step completion path | QA | Planned |
| REG-004 | Booking | User asks for demo | Slot offered and appointment created | QA | Planned |
| REG-005 | Escalation | Frustrated sentiment | Human handoff with transcript context | QA | Planned |

See `docs/roadmap.md` for implementation milestones and acceptance criteria.

