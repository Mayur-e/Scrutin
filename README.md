# Scrutin 🔍

> **Multi-agent AI misinformation verification platform.**
> Submit a claim or article URL — six independent AI agents investigate, cross-reference, and deliver a structured verdict you can trust.

[![Python](https://img.shields.io/badge/python-3.11%20%7C%203.12-blue)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-green)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-19-61DAFB)](https://react.dev/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

---

## Architecture

```
┌─────────────────────────────┐     HTTP POST /api/verify     ┌──────────────────────────────┐
│   React 19 + Vite Frontend  │ ───────────────────────────▶ │   Python FastAPI Backend      │
│   (port 5173 in dev)        │                               │   (port 8000)                 │
│                             │ ◀─── VerificationReport ───── │                               │
│  · Hero input               │                               │  · Decomposition Agent        │
│  · SpatialScroll panels     │                               │  · Evidence Agent             │
│  · Verification workspace   │                               │  · Credibility Agent          │
└─────────────────────────────┘                               │  · Forensics Agent            │
                                                              │  · Adversarial Agent          │
                                                              │  · Orchestrator Agent         │
                                                              └──────────────────────────────┘
```

---

## Prerequisites

| Tool | Version | Install |
|---|---|---|
| Python | 3.11 or 3.12 | [python.org](https://www.python.org/) |
| Node.js | 20+ | [nodejs.org](https://nodejs.org/) |
| pnpm | latest | `npm install -g pnpm` |
| git | any | [git-scm.com](https://git-scm.com/) |

---

## Quick Start

### 1. Clone & enter the repo
```bash
git clone https://github.com/yourteam/scrutin.git
cd scrutin
```

### 2. Set up the Python backend

```bash
cd backend

# Create and activate virtual environment
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate

# Install all dependencies (includes FastAPI + uvicorn)
pip install -r requirements.txt

# Configure secrets
cp .env.example .env
# Edit .env and fill in:
#   GOOGLE_API_KEY, GROQ_API_KEY, SERPER_API_KEY, PINECONE_API_KEY

# Run DB migrations
python -m app.memory.migrations

# Start the API server (port 8000)
python -m app.server
```

The backend will be running at **http://localhost:8000**
- Swagger UI: http://localhost:8000/api/docs
- Health check: http://localhost:8000/api/healthz

### 3. Set up the React frontend

```bash
cd frontend

# Install dependencies
pnpm install

# Configure the API URL
cp artifacts/scrutin/.env.example artifacts/scrutin/.env.local
# .env.local already has: VITE_API_URL=http://localhost:8000

# Start the Vite dev server (port 5173)
cd artifacts/scrutin
pnpm dev
```

The frontend will be at **http://localhost:5173**

---

## Using Docker Compose (Alternative)

```bash
# From repo root — starts both services
docker-compose up

# Frontend: http://localhost:5173
# Backend:  http://localhost:8000
```

> **Note:** You still need to create `backend/.env` with your API keys before running Docker.

---

## API Reference

### `POST /api/verify`

Run a full verification on a claim or URL.

**Request body:**
```json
{
  "claim": "The Eiffel Tower was built in 1889"
}
```
or
```json
{
  "url": "https://example.com/article"
}
```

**Response:** `VerificationReport`
```json
{
  "run_id": "a1b2c3",
  "overall_verdict": "true",
  "credibility_score": 85.0,
  "confidence": 0.92,
  "iterations_used": 3,
  "processing_time_seconds": 5.4,
  "evidence_used": [...],
  "adversarial_summary": "...",
  "budget_exhausted": false
}
```

### `GET /api/healthz`
Returns `{ "status": "ok" }` when the server is running.

---

## CLI Usage (Backend Only)

```bash
cd backend
# Activate venv first

# Verify a text claim
python -m app.cli verify --claim "The Eiffel Tower was built in 1889" --trace

# Verify a URL
python -m app.cli verify --url "https://example.com/article"

# Run regression test suite
python -m app.cli test

# Show database statistics
python -m app.cli stats
```

---

## Project Structure

```
scrutin/
├── backend/                        # Python multi-agent engine
│   ├── app/
│   │   ├── api.py                  # ← FastAPI HTTP server (NEW)
│   │   ├── server.py               # ← Uvicorn runner (NEW)
│   │   ├── cli.py                  # Original CLI interface
│   │   ├── orchestrator/           # Orchestration loop
│   │   ├── agents/                 # Six AI agents
│   │   ├── protocols/              # Blackboard + message types
│   │   ├── memory/                 # SQLite + Pinecone memory
│   │   └── tools/                  # Web search, WHOIS, etc.
│   ├── requirements.txt
│   └── .env.example
│
└── frontend/                       # TypeScript / React workspace
    ├── lib/
    │   ├── api-spec/
    │   │   └── openapi.yaml        # ← Extended with /verify (MODIFIED)
    │   ├── api-client-react/       # Generated React Query hooks
    │   └── api-zod/                # Generated Zod validators
    └── artifacts/
        ├── api-server/             # Express stub (unused in integration)
        └── scrutin/                # Main React app
            └── src/
                ├── hooks/
                │   └── useVerification.ts  # ← New API hook (NEW)
                └── lib/
                    └── api-config.ts       # ← Sets API base URL (NEW)
```

---

## License

MIT — see [LICENSE](LICENSE)
