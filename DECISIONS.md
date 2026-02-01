# DECISIONS.md

Design decisions, priorities, and simplifications for the Lender Matching Platform.

---

## Lender Requirements Prioritized

We modeled lender credit policies around the criteria that appear most often in the provided PDFs and in typical equipment finance guidelines:

1. **FICO score** – Min/max or tiered minimums per program. Always checked when the lender defines it.
2. **PayNet (MasterScore)** – Min/max when present. Handled 0–100 scale and documented 3-digit OCR artifacts (e.g. 660 → 66) in the parser.
3. **Loan amount** – Min/max required for every program. Used as a hard gate for eligibility.
4. **Time in business (TIB)** – Minimum years. Common in all five PDFs.
5. **Geographic** – Allowed or excluded states (2-letter codes). Extracted from phrases like “does not lend in CA, NV.”
6. **Industry** – Allowed or excluded industries (e.g. Trucking). Kept separate from equipment.
7. **Equipment** – Allowed/excluded types, max equipment age. Separated from industry so “no Aircraft” is equipment, “no Trucking” is industry.
8. **Min revenue** – Single minimum annual revenue when stated in the guidelines.

**Custom rules** – Unmodeled constraints (e.g. “US Citizen only,” “No BK in last 7 years”) are stored as `custom_rules` (name + description) so they appear in the UI and can be enforced manually or extended later. We did not build an expression engine for them.

---

## Simplifications Made and Why

- **Database: SQLite by default** – Assignment recommends PostgreSQL. We support PostgreSQL via `DATABASE_URL` but default to SQLite for zero-friction local dev. Production can use PostgreSQL; schema and code are the same.
- **No Hatchet** – Workflow orchestration was marked optional. We use a single async underwriting function (load application + lenders, evaluate, persist). With more time we would consider Hatchet for retries, parallel lender evaluation, and auditability.
- **Underwriting is synchronous from the API’s perspective** – The client POSTs to underwrite and gets the run record; the run is completed in the same request. We do not use a job queue or background worker. For large lender sets, we would move to async jobs and polling.
- **PDF parsing: LLM optional** – We support OpenAI and Gemini for higher-quality extraction when API keys are set; otherwise we use deterministic regex/heuristics so the system works without keys. No Groq; the doc mistakenly said “Groq” — order is Gemini first, then OpenAI.
- **Seed script** – `scripts/seed_lenders.py` only ensures DB tables exist. Lenders are created via the API (or UI), including from parsed PDFs, not from hardcoded data in the repo.
- **Application “completeness”** – We rely on the application form and Pydantic validation. We added a lightweight check before underwriting: if critical fields for matching (e.g. FICO, loan amount) are missing, the API returns 400 with a clear message. We did not implement a separate “completeness” score or multi-step validation workflow.

---

## What We Would Add With More Time

- **Tests** – Broader coverage: API integration tests (FastAPI TestClient), frontend critical-path tests, and more matching-engine edge cases (tiered FICO, geographic edge cases, multiple programs per lender).
- **Workflow / jobs** – Background underwriting runs with polling or webhooks; optional Hatchet (or similar) for retries and parallelization.
- **Audit and versioning** – Version lender criteria and application snapshots when a run is started, so results are reproducible and auditable.
- **Custom rules engine** – Interpret a subset of `custom_rules` (e.g. simple expressions or tags) so some unmodeled rules can be auto-evaluated.
- **API auth** – No auth in this submission; we would add API keys or OAuth for production.
- **PostgreSQL as default in prod** – Document and default `DATABASE_URL` to PostgreSQL in production; keep SQLite for local dev only.
