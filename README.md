# FHH AI Optimizer

Predictive maintenance + demand forecasting for Fine Hygienic Holding's
Valmet DCT 200TS tissue lines. Built for the May 15 panel pitch as the
Malia Group's AI initiative deliverable.

## Live URLs

| Service | URL | Purpose |
|---|---|---|
| Frontend (Vercel) | https://fhh-ai-optimizer.vercel.app | React dashboard — what panelists see |
| Backend (Railway) | https://fhh-ai-optimizer-production.up.railway.app | FastAPI + ML + Anthropic — 24 endpoints |
| API Contract | [docs/API_CONTRACT-2.md](docs/API_CONTRACT-2.md) | Endpoint specifications |
| Health check | https://fhh-ai-optimizer-production.up.railway.app/health | Railway liveness probe |
| API docs (Swagger) | https://fhh-ai-optimizer-production.up.railway.app/docs | Interactive endpoint explorer |

## What's inside

- **`backend/`** — FastAPI service exposing the 24 contract endpoints
  across three modules (Module 1: maintenance, Module 2: demand,
  Module 3: chat) plus cross-cutting KPIs. Predictive maintenance
  uses a RandomForest trained on simulated Valmet sensor data with
  five categories of realistic noise injected; demand forecasting
  uses Prophet with Ramadan/Eid/back-to-school holiday calendars;
  chat is Claude (sonnet-4-6) with the other endpoints exposed as
  tools.
- **`frontend/app/`** — React + Vite dashboard. Three pages
  (overview, machine detail, alerts) plus a persistent chat sidebar.
  Wired to the backend via `VITE_API_URL`.
- **`docs/`** — API contract (locked source of truth), database
  architecture rationale, machine + sensor reference. Treat
  `docs/API_CONTRACT-2.md` as immutable spec.

## Local setup

### Backend

```bash
cd backend
pip install -r requirements.txt
uvicorn ai_model.api:app --reload --port 8000
# → http://127.0.0.1:8000  (Swagger at /docs)
```

For the chat endpoint, set `ANTHROPIC_API_KEY` in `backend/.env`
(see `backend/.env.example`).

### Frontend

```bash
cd frontend/app
cp .env.example .env  # then edit if pointing at local backend
npm install
npm run dev
# → http://127.0.0.1:5173
```

## Architecture in one diagram

```
Valmet DNA DCS (real, in production)
        │
        ▼ per-minute sensor readings
backend/timescale/sensor_simulator.py  ←  raw layer (2.1M rows/yr)
        │
        ▼ hourly aggregation + feature engineering
backend/timescale/etl.py               ←  features.parquet (47 cols)
        │
        ▼ supervised + unsupervised training
backend/ai_model/{train,predict,evaluate}_model.py  ←  .pkl artifacts
        │
        ▼ exposed as HTTP endpoints
backend/ai_model/api.py                ←  FastAPI (24 routes)
        │
        ▼ React + chat (Claude with API as tools)
frontend/app/                          ←  Vercel dashboard
```

Synthetic data is deterministically regenerable from
`python backend/timescale/sensor_simulator.py --yearly-raw` (seed=42).
A clean-baseline model checkpoint lives at
`backend/ai_model/artifacts/clean_baseline/` as a safety fallback.

## Status

Phase 2 complete: 24/24 contract endpoints live, model evaluated under
five categories of realistic noise (Precision 0.985, Recall 0.915
on time-based holdout), backend deployed to Railway, frontend
deployed to Vercel. Next: panel pitch on May 15.

# FHH AI Optimizer

An end-to-end AI platform for industrial operations, built for four manufacturing
lines across the UAE, Egypt and Jordan.

## What it does

- **Predictive maintenance** — flags likely equipment failures before they interrupt
  a production line
- **Demand forecasting** — projects production demand from historical operating data
- **Decision-support assistant** — a grounded LLM interface that answers operational
  questions against the platform's own data rather than generating free text

## Stack

| Layer | Technology |
| --- | --- |
| Backend | FastAPI (Python), REST API |
| Frontend | React |
| Database | PostgreSQL |
| Machine learning | scikit-learn, Prophet |
| Deployment | Railway, Vercel, Git-based CI/CD |

## Architecture

The platform is a FastAPI service over PostgreSQL serving a React front end.
Forecasting and maintenance models are trained offline and served through the same
API. The assistant layer sits on top of the API rather than querying the database
directly, so every answer it gives is backed by a real endpoint response.

## Status

Built April–May 2026 as sole developer, owning the work from data architecture
through to production deployment, and presented to the client's senior management.
