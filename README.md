# Lender Matching Platform

Loan underwriting and lender matching: evaluate business loan applications against multiple lenders' credit policies, with a normalized policy model and PDF-based onboarding of new lenders.

## Quick Start

1. **Backend** (port 3005):
   ```bash
   cd backend
   python3 -m venv .venv && source .venv/bin/activate
   pip install -r requirements.txt
   cp .env.example .env
   python run.py
   ```
   See [backend/README.md](backend/README.md) for database options and API keys.

2. **Frontend** (port 3000):
   ```bash
   cd frontend
   npm install
   npm run dev
   ```
   Set `NEXT_PUBLIC_API_URL=http://localhost:3005` if the API is not on that URL. See [frontend/README.md](frontend/README.md).

3. Open [http://localhost:3000](http://localhost:3000): create an application, run underwriting, view match results and lender policies.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  Frontend (Next.js + TypeScript)                                │
│  Applications · Application detail · Results · Lender policies  │
└────────────────────────────┬────────────────────────────────────┘
                             │ REST (camelCase)
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  Backend (FastAPI)                                               │
│  api/          applications, lenders, underwriting              │
│  services/     matching_engine, run_underwriting                 │
│  pdf_ingestion parser (LLM or regex) → suggested criteria      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  Database (SQLite default / PostgreSQL)                         │
│  loan_applications · lenders · lender_programs · underwriting_runs│
└─────────────────────────────────────────────────────────────────┘
```

- **Matching flow**: User runs underwriting → backend loads application + all lenders (with programs) → for each lender, `matching_engine.evaluate_application()` computes eligibility, best program, fit score, rejection reasons and per-criterion results → results persisted on the run and returned to the frontend.
- **Adding lenders**: Upload a guideline PDF via UI (or API `POST /api/lenders/parse-pdf`), review/edit suggested name, slug, and criteria, then create lender (and programs) via API/UI. No hardcoded seed data.

## Repo Structure

- **backend/** – FastAPI app, models, matching engine, PDF parser. [README](backend/README.md) · [API docs at /docs](http://localhost:3005/docs) when server is running.
- **frontend/** – Next.js app (applications, results, lenders). [README](frontend/README.md)
- **DECISIONS.md** – Design choices, simplifications, and future work.
# lender-matching-platform
