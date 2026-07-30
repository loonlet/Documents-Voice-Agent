# API Reference

Repository audit result: no backend API implementation files are present. This document provides:

1) what is actually evidenced in the repository,
2) a production API specification to implement next.

## 1) APIs Found in Code (Current State)

| Type | Endpoint/Interface | Status | Evidence |
|---|---|---|---|
| Lead capture webhook | Not defined (planned) | Not Implemented | `src/AIVOICEAGE_board.html` (`AIVOICEAGE-19`) |
| Outbound call trigger | Not defined (planned) | Not Implemented | `src/AIVOICEAGE_board.html` (`AIVOICEAGE-20`) |
| Call retry/fallback service | Not defined (planned) | Not Implemented | `src/AIVOICEAGE_board.html` (`AIVOICEAGE-25`) |
| Calendar booking integration | Not defined (planned) | Not Implemented | `src/AIVOICEAGE_board.html` (`AIVOICEAGE-24`) |

No concrete HTTP routes, request/response schemas, or OpenAPI files were found in repository code.

## 2) Proposed Core API (To Be Implemented) **[ASSUMPTION]**

Base path: `/api/v1`

### Health and readiness

- `GET /healthz`
- `GET /readyz`

### Leads

- `POST /leads/webhook`
  - Purpose: ingest abandoned-session lead from website/portal.
  - Auth: HMAC signature header.
- `GET /leads/{leadId}`
- `PATCH /leads/{leadId}`
- `POST /leads/{leadId}/qualify`

### Calls

- `POST /calls/trigger`
  - Purpose: enqueue outbound callback (30-60s SLA target).
- `GET /calls/{callId}`
- `GET /calls?status=&from=&to=&agentId=`
- `POST /calls/{callId}/retry`
- `POST /calls/{callId}/handoff`

### Appointments

- `GET /appointments/slots?from=&to=&timezone=`
- `POST /appointments`
- `PATCH /appointments/{appointmentId}`

### Agent configuration

- `GET /agent-config`
- `PUT /agent-config`

### Observability and audit

- `GET /metrics` (Prometheus)
- `GET /audit/events?leadId=&callId=`

## 3) Webhook Specifications **[ASSUMPTION]**

### 3.1 Lead Capture Webhook

`POST /api/v1/leads/webhook`

Headers:

- `X-Loonlet-Signature: sha256=<hex>`
- `X-Loonlet-Timestamp: <unix_epoch_ms>`
- `X-Idempotency-Key: <uuid>`

Request example:

```json
{
  "event": "lead.abandoned",
  "lead": {
    "external_id": "site-123",
    "name": "Jane Doe",
    "phone": "+15551234567",
    "email": "jane@example.com",
    "language_hint": "en",
    "missing_fields": ["company", "budget"],
    "context": {
      "page": "/pricing",
      "product": "enterprise-plan"
    }
  },
  "occurred_at": "2026-07-30T08:12:21Z"
}
```

Response example:

```json
{
  "accepted": true,
  "lead_id": "8d2a73d4-a40f-48e3-b223-d3b0ddd9940b",
  "queued_call_eta_sec": 30
}
```

### 3.2 Telephony Event Webhook

`POST /api/v1/webhooks/telephony/events`

Event examples: `call.started`, `call.answered`, `call.ended`, `call.failed`.

## 4) Tool/Model Internal APIs **[ASSUMPTION]**

- `POST /stt/stream` (Faster-Whisper wrapper service)
- `POST /llm/chat` (Ollama-compatible adapter)
- `POST /tts/synthesize` (Piper wrapper service)
- `POST /rag/query` (retrieval service)

## 5) Error Model **[ASSUMPTION]**

```json
{
  "error": {
    "code": "INVALID_SIGNATURE",
    "message": "Webhook signature verification failed",
    "request_id": "req_01J123..."
  }
}
```

Standard statuses:

- `200/201`: success
- `202`: async accepted
- `400`: validation error
- `401/403`: auth/authz failure
- `409`: duplicate idempotency key
- `429`: rate-limited
- `500`: internal failure

## 6) SLA and Retry Semantics **[ASSUMPTION]**

- Callback enqueue within `<= 5s` of lead webhook acceptance.
- First outbound dial attempt target: `30-60s` after lead event.
- Retry policy example: 3 attempts, exponential backoff, max 24h campaign window.

## 7) OpenAPI and Contract Testing Plan

- Maintain `openapi.yaml` as source of truth.
- Generate typed clients for dashboard and integration workers.
- Add consumer-driven contract tests for CRM, WhatsApp, and email providers.

Related docs: `docs/setup.md`, `docs/security.md`, `docs/testing.md`.

