# Setup and Deployment

Current repository state: no runnable backend services are committed yet. This setup guide is a production-oriented specification aligned to the backlog in `src/AIVOICEAGE_board.html` and architecture in `docs/architecture.md`.

## 1) Local Development (Docker Compose)

## Prerequisites

- Docker Engine + Docker Compose plugin
- NVIDIA Container Toolkit (only if running GPU-accelerated STT/LLM locally)
- Linux host recommended for telephony stack tests **[ASSUMPTION]**

## Suggested local service set

- `freeswitch`
- `agent-orchestrator` (Python/Pipecat)
- `api` (FastAPI webhook + integrations)
- `ollama`
- `faster-whisper`
- `piper`
- `postgres`
- `redis`
- `minio`
- `otel-collector`, `prometheus`, `grafana`, `loki`

## Example `docker-compose.yml` skeleton **[ASSUMPTION]**

```yaml
version: "3.9"
services:
  api:
    image: loonlet/api:dev
    env_file: .env
    depends_on: [postgres, redis]
    ports: ["8080:8080"]

  agent-orchestrator:
    image: loonlet/orchestrator:dev
    env_file: .env
    depends_on: [api, ollama, faster-whisper, piper]

  ollama:
    image: ollama/ollama:latest
    ports: ["11434:11434"]
    volumes: ["ollama:/root/.ollama"]

  faster-whisper:
    image: ghcr.io/your-org/faster-whisper:latest
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]

  piper:
    image: rhasspy/piper:latest

  freeswitch:
    image: signalwire/freeswitch:latest
    network_mode: host

  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: loonlet
      POSTGRES_USER: loonlet
      POSTGRES_PASSWORD: loonlet
    volumes: ["pgdata:/var/lib/postgresql/data"]

  redis:
    image: redis:7-alpine

  minio:
    image: minio/minio:latest
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: minio123
    ports: ["9000:9000", "9001:9001"]

volumes:
  pgdata: {}
  ollama: {}
```

## Local boot commands **[ASSUMPTION]**

```powershell
docker compose pull
docker compose up -d
```

## 2) Cloud Deployment Patterns

## A) Single-VM (Small Business)

- Deploy all services on one hardened VM.
- Use one GPU if running self-hosted STT+LLM.
- Keep SIP and HTTPS on separate interfaces/VLANs **[ASSUMPTION]**.
- Use daily backups for PostgreSQL and object storage.

## B) Multi-Node Kubernetes (High Call Volume)

- Node pools:
  - CPU pool: API, orchestration, telephony control-plane
  - GPU pool A: STT (Faster-Whisper)
  - GPU pool B: LLM (Ollama)
  - Optional CPU/GPU pool C: TTS
- Deploy with Helm charts per service.
- Use HPA on queue depth and concurrent call count.
- Use PodDisruptionBudgets and anti-affinity for HA.

## Kubernetes baseline resources **[ASSUMPTION]**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: loonlet-orchestrator
spec:
  replicas: 3
  selector:
    matchLabels:
      app: loonlet-orchestrator
  template:
    metadata:
      labels:
        app: loonlet-orchestrator
    spec:
      containers:
        - name: orchestrator
          image: loonlet/orchestrator:latest
          envFrom:
            - secretRef:
                name: loonlet-secrets
          resources:
            requests:
              cpu: "500m"
              memory: "1Gi"
            limits:
              cpu: "2000m"
              memory: "4Gi"
```

## GPU sizing guidance **[ASSUMPTION]**

- Pilot (up to ~10 concurrent calls): 1 GPU (L4/A10 class), 16 vCPU, 64 GB RAM.
- Growth (10-50 concurrent calls): 2-4 GPUs split STT and LLM.
- Enterprise (>50 concurrent calls): dedicated STT and LLM node pools + autoscaling.

## 3) Configuration and Secrets

- Store secrets in Vault or Kubernetes Secrets encrypted with SOPS.
- Keep separate keys for SIP auth, DB, JWT signing, SMTP, WhatsApp, CRM.
- Rotate credentials every 90 days **[ASSUMPTION]**.

## 4) Minimum Environment Variables **[ASSUMPTION]**

```env
APP_ENV=production
API_BASE_URL=https://api.example.com
POSTGRES_DSN=postgresql://...
REDIS_URL=redis://...
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=...
MINIO_SECRET_KEY=...
OLLAMA_URL=http://ollama:11434
WHISPER_URL=http://faster-whisper:8000
PIPER_URL=http://piper:5000
JWT_ISSUER=loonlet
JWT_AUDIENCE=loonlet-dashboard
SIP_DOMAIN=sip.example.com
SIP_USERNAME=...
SIP_PASSWORD=...
```

## 5) Go-Live Checklist

- Telephony: SIP trunk registered, dialplan tested for inbound/outbound/error branches.
- AI loop: STT/LLM/TTS streaming validated under load.
- Integrations: CRM + booking + notifications e2e tested.
- Security: TLS, SRTP/TLS-SIP, secret rotation, RBAC.
- Compliance: call-consent prompts and retention policy active.
- Observability: p95 latency, call success rate, callback SLA alerts configured.

For hardening and compliance controls, see `docs/security.md`.


