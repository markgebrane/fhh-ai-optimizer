# FHH AI Optimizer

Predictive maintenance and demand forecasting platform built for Fine Hygienic
Holding's Valmet DCT 200TS tissue lines, delivered as the Malia Group's AI
initiative. Built solo, from data architecture through production deployment,
and presented to the client's senior management at the May 15 panel.

## Live URLs

| Service | URL | Purpose |
|---|---|---|
| Frontend (Vercel) | https://fhh-ai-optimizer.vercel.app | React dashboard shown to panelists |
| Backend (Railway) | https://fhh-ai-optimizer-production.up.railway.app | FastAPI + ML + Anthropic, 24 endpoints |
| API Contract | [docs/API_CONTRACT-2.md](docs/API_CONTRACT-2.md) | Endpoint specifications |
| Health check | https://fhh-ai-optimizer-production.up.railway.app/health | Railway liveness probe |
| API docs (Swagger) | https://fhh-ai-optimizer-production.up.railway.app/docs | Interactive endpoint explorer |

## What it does

- **Predictive maintenance** flags likely equipment failures before they
  interrupt a production line.
- **Demand forecasting** projects production demand from historical operating
  data.
- **Decision-support assistant** is a grounded LLM interface that answers
  operational questions against the platform's own data instead of
  generating free text.

## What's inside

- **`backend/`**, a FastAPI service exposing 24 contract endpoints across
  three modules: maintenance, demand, chat, plus cross-cutting KPIs.
  Predictive maintenance uses a RandomForest trained on simulated Valmet
  sensor data with five categories of realistic noise injected. Demand
  forecasting uses Prophet with Ramadan/Eid/back-to-school holiday calendars.
  Chat is Claude (sonnet-4-6) with the other endpoints exposed as tools.
- **`frontend/app/`**, a React + Vite dashboard: overview, machine detail and
  alerts pages, plus a persistent chat sidebar, wired to the backend via
  `VITE_API_URL`.
- **`docs/`**, the API contract (locked source of truth), database
  architecture rationale, and machine/sensor reference. Treat
  `docs/API_CONTRACT-2.md` as immutable spec.

## Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI (Python), REST API |
| Frontend | React + Vite |
| Database | PostgreSQL / TimescaleDB |
| Machine learning | scikit-learn, Prophet |
| Deployment | Railway, Vercel, Git-based CI/CD |

## Architecture

```
Valmet DNA DCS (real, in production)
        |
        v  per-minute sensor readings
backend/timescale/sensor_simulator.py   <- raw layer (2.1M rows/yr)
        |
        v  hourly aggregation + feature engineering
backend/timescale/etl.py                <- features.parquet (47 cols)
        |
        v  supervised + unsupervised training
backend/ai_model/{train,predict,evaluate}_model.py  <- .pkl artifacts
        |
        v  exposed as HTTP endpoints
backend/ai_model/api.py                 <- FastAPI (24 routes)
        |
        v  React + chat (Claude with API as tools)
frontend/app/                           <- Vercel dashboard
```

Synthetic data is deterministically regenerable via
`python backend/timescale/sensor_simulator.py --yearly-raw` (seed=42). A
clean-baseline model checkpoint lives at
`backend/ai_model/artifacts/clean_baseline/` as a safety fallback.

## Local setup

### Backend

```bash
cd backend
pip install -r requirements.txt
uvicorn ai_model.api:app --reload --port 8000
# -> http://127.0.0.1:8000  (Swagger at /docs)
```

For the chat endpoint, set `ANTHROPIC_API_KEY` in `backend/.env` (see
`backend/.env.example`).

### Frontend

```bash
cd frontend/app
cp .env.example .env  # then edit if pointing at local backend
npm install
npm run dev
# -> http://127.0.0.1:5173
```

## Status

Phase 2 complete: 24/24 contract endpoints live, model evaluated under five
categories of realistic noise (Precision 0.985, Recall 0.915 on a time-based
holdout), backend deployed to Railway, frontend deployed to Vercel.

## License

MIT. See [LICENSE](LICENSE).
