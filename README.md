# Loonlet - AI Voice Agent for Sales and Customer Support

Loonlet is intended to be a 24/7 AI voice agent that answers business phone calls, understands customer intent, qualifies leads, books appointments, and escalates to humans when needed.

This repository currently contains:

- product/solution narrative (`src/loonlet-voice-ai.html`)
- implementation backlog board (`src/AIVOICEAGE_board.html`)
- a placeholder Java template (`src/Main.java`)

It does **not** yet contain a production backend implementation of the voice platform.

## Problem Statement

Businesses lose high-intent leads and support quality when phone systems rely on static IVR menus, delayed callbacks, and disconnected systems. Loonlet aims to provide low-latency, context-aware, multilingual voice automation with strict security and operational controls.

## Key Feature Goals

- Real-time STT -> LLM -> TTS conversational loop
- Telephony integration (FreeSWITCH + SIP + dialplan + streaming)
- Lead capture webhook + outbound callback SLA (30-60s)
- Sales/support conversation flows (FAQ, missing info, purchase nudge)
- Appointment/demo booking
- Human handoff with context
- Full observability, security hardening, and compliance controls

Current implementation status of each goal is tracked in `docs/features.md`.

## Documentation Index

- Feature audit first: `docs/features.md`
- Architecture and diagrams: `docs/architecture.md`
- Open-source tech choices: `docs/tech-stack.md`
- Setup and deployment patterns: `docs/setup.md`
- Security and compliance baseline: `docs/security.md`
- API and webhook specification: `docs/api-reference.md`
- Testing and QA strategy: `docs/testing.md`
- Gap-to-delivery roadmap: `docs/roadmap.md`

## Quick Start

Because runtime services are not yet implemented in this repo, quick start currently means reviewing the artifacts and planning docs:

```powershell
# Open project artifacts in browser
# (adjust path if your shell or editor differs)
start src\AIVOICEAGE_board.html
start src\loonlet-voice-ai.html
```

## Current State Snapshot

- Implemented evidence: environment/account setup planning and project board UI.
- Partial evidence: local STT/LLM/TTS loop claims and FreeSWITCH setup in progress (ticket-level only).
- Not implemented in code: telephony runtime, webhook/API services, integrations, dashboard product UI, RAG, and formal QA.

See `docs/roadmap.md` for implementation-ready specifications and acceptance criteria.


