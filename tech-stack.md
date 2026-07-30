# Tech Stack (Open-Source and Self-Hostable)

This file lists components referenced in the repository plus OSS-only production recommendations.

## Current References in Repository

| Component | Seen In | Open Source | Self-Hostable | Notes |
|---|---|---:|---:|---|
| Python 3.10+ | `src/AIVOICEAGE_board.html` (`AIVOICEAGE-1`) | Yes | Yes | Mentioned as runtime target, no code committed. |
| Pipecat (`pipecat-ai`) | `src/AIVOICEAGE_board.html` (`AIVOICEAGE-1`) | Yes | Yes | Orchestration framework mentioned in setup ticket. |
| FreeSWITCH | `src/AIVOICEAGE_board.html` (`AIVOICEAGE-5`) | Yes | Yes | Planned telephony core. |
| Silero VAD | `src/AIVOICEAGE_board.html` (`AIVOICEAGE-4`) | Yes | Yes | Planned VAD model. |
| Groq API | `src/AIVOICEAGE_board.html` (`AIVOICEAGE-1`, `-2`) | No | No | Not compliant with strict OSS-only production requirement. |
| Cartesia/Baseten TTS | `src/AIVOICEAGE_board.html` (`AIVOICEAGE-1`, `-3`) | No | No | Not compliant with strict OSS-only production requirement. |
| ngrok / Cloudflare Tunnel | `src/AIVOICEAGE_board.html` (`AIVOICEAGE-1`) | No / Mixed | No | Acceptable for local testing only. |

## Recommended OSS-Only Production Stack

| Layer | Component | Recommended Version* | License | Why Chosen |
|---|---|---|---|---|
| Telephony PBX | FreeSWITCH | `1.10.x` **[ASSUMPTION]** | MPL-1.1 | Mature SIP stack, call control, recording hooks, ESL integration. |
| SBC | Kamailio or OpenSIPS | `5.8.x` / `3.5.x` **[ASSUMPTION]** | GPLv2 | SIP edge hardening, anti-fraud, NAT traversal, rate limiting. |
| Voice Orchestration | Pipecat | latest stable **[ASSUMPTION]** | BSD-2-Clause **[ASSUMPTION]** | Realtime pipeline composition for STT/LLM/TTS stages. |
| LLM Runtime | Ollama | `0.3.x+` **[ASSUMPTION]** | MIT | Local model serving with streaming support. |
| LLM Models | Llama 3.1 / Mistral 7B Instruct | model-specific **[ASSUMPTION]** | OSS model licenses | Good quality-latency balance for call-center workflows. |
| STT Runtime | Faster-Whisper (CTranslate2) | latest stable **[ASSUMPTION]** | MIT | High-throughput local transcription with GPU acceleration. |
| STT Models | Whisper small/medium | model-specific **[ASSUMPTION]** | MIT | Widely adopted multilingual ASR baseline. |
| TTS Runtime | Piper | latest stable **[ASSUMPTION]** | MIT | Fully local low-latency speech synthesis. |
| VAD | Silero VAD | latest stable **[ASSUMPTION]** | MIT **[ASSUMPTION]** | Low-latency speech boundary detection, supports barge-in. |
| API Backend | FastAPI + Uvicorn | `0.115.x` / `0.30.x` **[ASSUMPTION]** | MIT | Lightweight async APIs and webhook endpoints. |
| Task Queue | Celery/RQ + Redis | `5.4.x` / `7.x` **[ASSUMPTION]** | BSD / BSD | Outbound retry scheduling and SLA control loops. |
| Primary DB | PostgreSQL | `16.x` **[ASSUMPTION]** | PostgreSQL | Reliable relational store for leads/calls/appointments. |
| Object Storage | MinIO | latest stable **[ASSUMPTION]** | AGPLv3 | Self-hosted recordings and artifacts. |
| RAG Vector Store | Qdrant or pgvector | latest stable **[ASSUMPTION]** | Apache-2.0 / PostgreSQL | Hybrid search-ready retrieval for FAQ/policy grounding. |
| Embeddings | BGE/E5 open models | model-specific **[ASSUMPTION]** | OSS model licenses | Strong semantic retrieval performance. |
| Observability | OpenTelemetry Collector | `0.1xx` **[ASSUMPTION]** | Apache-2.0 | Unified traces/metrics/log export. |
| Metrics | Prometheus | `2.5x` **[ASSUMPTION]** | Apache-2.0 | Industry standard metrics/alerting backend. |
| Dashboards | Grafana | `11.x` **[ASSUMPTION]** | AGPLv3 | Real-time ops and SLA dashboards. |
| Logs | Loki | `3.x` **[ASSUMPTION]** | AGPLv3 | Label-based log aggregation. |
| Secrets | HashiCorp Vault or SOPS + age | latest stable **[ASSUMPTION]** | BSL / MPL-2.0 | Centralized secret lifecycle and rotation. |
| Container Runtime | Docker + Compose | latest stable **[ASSUMPTION]** | Apache-2.0 | Local deployment baseline. |
| Orchestration | Kubernetes + Helm | `1.30+` **[ASSUMPTION]** | Apache-2.0 | Multi-node scaling and HA operations. |

\* Versions are recommended pins because the repository does not include dependency manifests.

## Components to Remove/Replace for OSS-Only Compliance

| Non-OSS or managed dependency | Replace With | Migration Notes |
|---|---|---|
| Groq LLM API | Ollama + local Llama/Mistral | Planned in `AIVOICEAGE-15`. |
| Groq Whisper API | Faster-Whisper | Planned in `AIVOICEAGE-16`. |
| Cartesia/Baseten TTS | Piper | Planned in `AIVOICEAGE-17`. |
| ngrok/Cloudflare for production ingress | Self-managed reverse proxy + VPN/SBC | Keep tunnels only for local testing. |

## Integration Targets (OSS-first)

- CRM: EspoCRM, Twenty CRM, or SuiteCRM.
- WhatsApp messaging: Chatwoot plus open WhatsApp connector (Cloud API-compatible gateway, self-hosted where policy allows).
- Email: Postal or Mailtrain with self-hosted SMTP.
- Calendar: CalDAV-compatible backend or self-hosted Cal.com.

See `docs/roadmap.md` for acceptance criteria and rollout stages.

