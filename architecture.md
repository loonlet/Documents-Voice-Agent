# Loonlet Architecture

This document reconciles repository evidence with a production target architecture.

- Current-state evidence is in `docs/features.md`.
- The diagrams below show the recommended OSS-only production architecture because most runtime components are not yet implemented.
- Any behavior not directly evidenced in code is marked **[ASSUMPTION]**.

## 1) System Component Architecture (Target)

```mermaid
flowchart LR
    Customer[(Customer Caller)]
    Admin[(Business Admin)]

    subgraph Telephony[Telephony Layer]
      SIP[SIP Trunk Provider\n(Asterisk/OpenSIPS-compatible)]
      FS[FreeSWITCH Cluster\nInbound/Outbound/Dialplan]
      RTP[RTP Audio Stream Bridge]
    end

    subgraph Voice[AI Voice Pipeline]
      VAD[Silero VAD\nTurn-taking/Barge-in]
      STT[Faster-Whisper Service\nStreaming STT]
      ORCH[Agent Orchestrator\nPipecat/Python Service]
      LLM[Ollama Inference\nLlama/Mistral]
      TOOLS[Tool Router\nCRM/Booking/Notifications]
      TTS[Piper TTS\nStreaming Synthesis]
    end

    subgraph RAG[RAG Knowledge Base]
      INGEST[Document Ingestion Worker]
      EMB[Embedding Model\n(BGE/E5)]
      VDB[(Qdrant/pgvector)]
      KB[(Product/Policy Docs)]
    end

    subgraph Data[Integrations and Data]
      API[Core API Service]
      WEBHOOK[Lead Webhook Receiver]
      CRM[(CRM: EspoCRM/Twenty/SuiteCRM)]
      CAL[(Calendar Service)]
      WA[(WhatsApp Gateway\nChatwoot + WA Connector)]
      EMAIL[(Postal/Mailtrain SMTP Service)]
      DB[(PostgreSQL)]
      OBJ[(S3/MinIO Recordings)]
      CACHE[(Redis Queue)]
    end

    subgraph Ops[Observability and Security]
      OTel[OpenTelemetry Collector]
      Prom[Prometheus]
      Graf[Grafana]
      Loki[Loki Logs]
      Vault[Vault/SOPS Secrets]
    end

    Admin --> API
    Customer --> SIP --> FS --> RTP --> VAD --> STT --> ORCH
    ORCH --> LLM
    ORCH --> TOOLS
    TOOLS --> API
    API --> CRM
    API --> CAL
    API --> WA
    API --> EMAIL
    API --> DB
    API --> CACHE
    API --> OBJ
    ORCH --> TTS --> RTP --> FS --> SIP --> Customer

    ORCH --> VDB
    INGEST --> EMB --> VDB
    KB --> INGEST

    FS --> OTel
    API --> OTel
    ORCH --> OTel
    OTel --> Prom --> Graf
    OTel --> Loki
    Vault --> API
    Vault --> ORCH
    Vault --> FS
```

## 2) End-to-End Call Flow Sequence (Inbound -> Qualification -> Booking -> Confirmation)

```mermaid
sequenceDiagram
    autonumber
    participant C as Customer
    participant SIP as SIP Trunk
    participant FS as FreeSWITCH
    participant STT as Faster-Whisper
    participant AG as Agent Orchestrator
    participant LLM as Ollama
    participant RAG as Qdrant/pgvector
    participant CRM as CRM
    participant CAL as Calendar API
    participant WA as WhatsApp Gateway
    participant EM as Email Service

    C->>SIP: Place inbound call
    SIP->>FS: Route INVITE
    FS->>AG: Start call session + stream audio
    AG->>STT: Streaming audio frames
    STT-->>AG: Partial/final transcript
    AG->>RAG: Retrieve FAQ/policy context
    RAG-->>AG: Top-k context chunks
    AG->>LLM: Prompt + context + tools
    LLM-->>AG: Reply + tool-call intent

    AG->>CRM: Lookup/create lead
    CRM-->>AG: Lead profile + missing fields
    AG->>FS: Play TTS prompt for missing info
    C->>FS: Provides profile details
    FS->>AG: Audio stream
    AG->>STT: Transcribe details
    STT-->>AG: Structured text
    AG->>CRM: Update lead fields

    AG->>CAL: Query availability **[ASSUMPTION]**
    CAL-->>AG: Candidate slots **[ASSUMPTION]**
    AG->>FS: Offer slots via TTS
    C->>FS: Confirms slot
    AG->>CAL: Create appointment **[ASSUMPTION]**
    CAL-->>AG: Booking confirmation **[ASSUMPTION]**

    AG->>WA: Send confirmation/opt-out text **[ASSUMPTION]**
    AG->>EM: Send confirmation email **[ASSUMPTION]**
    AG->>FS: Closing response + consent reminder
    FS-->>C: End call
```

## 3) Deployment Architecture (Single VM + Scaled Multi-Node)

```mermaid
flowchart TB
    subgraph Internet[Public Network]
      Caller[(Caller)]
      Admin[(Admin Browser)]
      SIPP[(SIP Provider)]
    end

    subgraph Edge[DMZ / Edge]
      LB[Ingress + TLS\n(NGINX/Traefik)]
      SBC[Session Border Controller\n(OpenSIPS/Kamailio) **[ASSUMPTION]**]
    end

    subgraph Small[Single-VM Deployment (SMB)]
      FS1[FreeSWITCH]
      API1[Core API + Webhook]
      AG1[Agent Orchestrator]
      STT1[Faster-Whisper]
      LLM1[Ollama]
      TTS1[Piper]
      DB1[(PostgreSQL)]
      RD1[(Redis)]
      OB1[(MinIO)]
      OBS1[OTel + Prometheus + Grafana + Loki]
    end

    subgraph Scale[Kubernetes Multi-Node (High Volume)]
      K8S[K8s Control Plane]
      FSPool[FreeSWITCH Pool]
      APIPool[API Pods]
      AGPool[Orchestrator Pods]
      STTPool[STT GPU Nodes]
      LLMPool[LLM GPU Nodes]
      TTSPool[TTS Nodes]
      Data[(Managed/Postgres HA + Redis + Object Storage)]
      ObsStack[OTel + Prometheus + Grafana + Loki + Alertmanager]
    end

    Caller --> SIPP --> SBC --> FS1
    Caller --> SIPP --> SBC --> FSPool
    Admin --> LB --> API1
    Admin --> LB --> APIPool

    FS1 --> AG1 --> STT1
    AG1 --> LLM1
    AG1 --> TTS1
    API1 --> DB1
    API1 --> RD1
    API1 --> OB1
    AG1 --> API1
    FS1 --> OBS1
    API1 --> OBS1
    AG1 --> OBS1

    FSPool --> AGPool --> STTPool
    AGPool --> LLMPool
    AGPool --> TTSPool
    APIPool --> Data
    AGPool --> APIPool
    FSPool --> ObsStack
    APIPool --> ObsStack
    AGPool --> ObsStack
```

## 4) Core Data Model (ER)

```mermaid
erDiagram
    CUSTOMER ||--o{ LEAD : owns
    CUSTOMER ||--o{ CALL : places
    LEAD ||--o{ CALL : drives
    LEAD ||--o{ APPOINTMENT : converts_to
    CALL ||--o{ APPOINTMENT : may_book
    AGENT_CONFIG ||--o{ CALL : governs

    CUSTOMER {
      uuid id PK
      string name
      string phone_e164
      string email
      string language_pref
      datetime created_at
      datetime updated_at
    }

    LEAD {
      uuid id PK
      uuid customer_id FK
      string source
      string status
      json missing_fields
      string consent_state
      datetime captured_at
      datetime updated_at
    }

    CALL {
      uuid id PK
      uuid customer_id FK
      uuid lead_id FK
      uuid agent_config_id FK
      string direction
      string disposition
      int duration_sec
      string recording_uri
      float stt_p95_ms
      float llm_p95_ms
      float tts_p95_ms
      datetime started_at
      datetime ended_at
    }

    APPOINTMENT {
      uuid id PK
      uuid lead_id FK
      uuid call_id FK
      string provider
      datetime slot_start
      datetime slot_end
      string status
      datetime created_at
      datetime updated_at
    }

    AGENT_CONFIG {
      uuid id PK
      string locale
      string voice_model
      string llm_model
      bool barge_in_enabled
      bool recording_enabled
      json escalation_rules
      datetime updated_at
    }
```

## 5) GPU and Performance Guidance

- Single VM baseline **[ASSUMPTION]**: 1 x NVIDIA L4/A10 class GPU, 16 vCPU, 64 GB RAM for low-to-medium concurrent calls.
- Scaled cluster **[ASSUMPTION]**: separate GPU pools for STT and LLM to isolate contention; keep TTS CPU-bound unless premium voices require GPU.
- Latency target **[ASSUMPTION]**: median turn latency under 1.5s and p95 under 2.5s for natural voice interaction.

For rollout priorities and acceptance criteria, see `docs/roadmap.md`. For operational controls, see `docs/security.md`.

