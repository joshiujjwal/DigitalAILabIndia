# CLAUDE.md — DigitalAILabIndia

Context for AI coding agents. Read this before touching any code.

---

## Quick Commands

```bash
# Install backend deps
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt -r requirements-dev.txt

# Run backend (dev, auto-reload)
uvicorn src.api.main:app --reload --port 8000

# Run all backend tests
pytest tests/ -v --cov=src --cov-report=term-missing

# Run a specific test file
pytest tests/unit/test_auth.py -v

# Lint + format
ruff check src/ tests/
ruff format src/ tests/

# Type check
mypy src/

# Run Celery worker (separate terminal)
celery -A src.workers.celery_app worker --loglevel=info

# Database migrations
alembic upgrade head               # apply all migrations
alembic revision --autogenerate -m "description"   # create migration
alembic downgrade -1               # roll back one

# Frontend (in frontend/)
npm run dev          # dev server on :5173
npm run test         # Vitest tests
npm run build        # production build
npm run lint         # ESLint
npm run typecheck    # tsc --noEmit

# Docker (infrastructure only — use local Python for dev)
docker compose up -d db redis minio
```

---

## Directory Map

```
src/
  api/
    main.py           ← FastAPI app factory, middleware, router mounting
    dependencies.py   ← get_current_user, require_role, get_db
    routers/
      auth.py         ← /auth/* endpoints
      datasets.py     ← /datasets/* endpoints
      inference.py    ← /inference/* endpoints
      finetuning.py   ← /finetune/* endpoints
      models.py       ← /models/* endpoints
      evaluation.py   ← /evaluate/* endpoints
  models/
    base.py           ← SQLAlchemy declarative base, UUIDMixin, TimestampMixin
    user.py           ← User ORM model
    dataset.py        ← Dataset ORM model
    model_registry.py ← ModelVersion ORM model
    fine_tuning_job.py← FineTuningJob ORM model
  services/
    auth.py           ← JWT, bcrypt, token helpers
    dataset_service.py← Business logic for dataset CRUD
    nlp_service.py    ← HuggingFace pipeline wrappers, model caching
    finetuning_service.py ← Job orchestration, config validation
    eval_service.py   ← Benchmark runners
  workers/
    celery_app.py     ← Celery app instance + config
    finetuning_worker.py ← Celery task for LoRA fine-tuning
    eval_worker.py    ← Celery task for benchmarking
  utils/
    config.py         ← Pydantic Settings (reads from .env)
    s3.py             ← MinIO/S3 client, presigned URL helpers
    schema_validator.py ← Dataset schema validation
    model_catalog.py  ← Hardcoded catalog of supported base models
    logging.py        ← structlog setup

tests/
  unit/               ← Pure unit tests, no DB, mock everything external
  integration/        ← Tests that hit a real test DB and Redis
  e2e/                ← Tests that hit the running FastAPI app via httpx

frontend/src/
  pages/              ← One file per route (DatasetHub, InferencePlayground, etc.)
  components/         ← Reusable UI pieces (DatasetCard, JobStatusBadge, etc.)
  hooks/              ← Custom hooks (useInferenceJob, useDatasets, etc.)
  services/api.ts     ← Axios instance + all typed API call functions
  types/index.ts      ← All shared TypeScript interfaces
```

---

## Non-Obvious Conventions

### Python / FastAPI
- **All UUIDs** — use `uuid.uuid4()`, stored as UUID type in Postgres (not VARCHAR)
- **Pydantic v2** throughout — use `model_validate()` not `.parse_obj()`, `model_dump()` not `.dict()`
- **Service layer is pure Python** — routers call services, services call models/utils. No SQLAlchemy queries in routers.
- **Dependency injection** for DB sessions: always use `Depends(get_db)` — never create sessions manually in routers
- **Async all the way** — all FastAPI route handlers are `async def`. Services that touch DB use `async with` session.
- **Error handling** — raise `HTTPException` only in routers. Services raise custom exceptions (`DatasetNotFoundError`, etc.) that routers catch and convert.
- **HuggingFace models** are loaded once at startup via `nlp_service.py` and stored in a module-level dict. Never reload per request.
- **Celery tasks** accept only JSON-serializable args (UUIDs as strings, not objects)

### Frontend
- **All API calls** go through `services/api.ts` — never use `fetch` directly in components
- **No `any` types** — always define proper TypeScript interfaces in `types/index.ts`
- **Error boundaries** around ML-heavy pages (inference, fine-tuning dashboard)
- **Indic font support** — use `font-family: 'Noto Sans', 'Noto Serif Devanagari', sans-serif` for text display areas that show Indian language content

### Testing
- **Unit tests use `pytest-mock`** to mock all external services (HuggingFace, S3, Celery)
- **Integration tests** use `pytest-asyncio` + `httpx.AsyncClient` against a real test DB
- **Test DB** — use a separate `test_digitalailabindia` Postgres database (set `TEST_DATABASE_URL` in `.env`)
- **Factory Boy** for test fixtures — see `tests/factories.py`
- **Never commit tests that `skip` without a reason** — skipped tests must have a linked TODO

### Indian Language Specifics
- Language codes follow ISO 639-1 (2-letter: `hi`, `ta`, `te`, etc.)
- Text direction: all current v0.1 languages are LTR except Urdu (`ur`) which is RTL — handle in frontend
- Indic numerals: normalize to ASCII digits before ML inference (use `unicodedata.normalize`)

---

## Workflow for AI Agents

1. **Start by running tests:** `pytest tests/ -v` — understand the current state before writing any code
2. **Pick a TODO item** from `TODO.md` that is in the current phase
3. **Write the test first** — get it to RED before writing implementation
4. **Implement** the minimum code to go GREEN
5. **Run full lint pass:** `ruff check src/ tests/ && mypy src/`
6. **Commit** with message format: `feat(<area>): <what>` or `fix(<area>): <what>`
7. **Update this file** if you discovered a new non-obvious convention

---

## Environment Variables (required)

See `.env.example` for full list. Critical ones:

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | Postgres connection string |
| `TEST_DATABASE_URL` | Separate test DB |
| `REDIS_URL` | Redis for Celery broker + result backend |
| `SECRET_KEY` | JWT signing key (min 32 chars, random) |
| `HF_TOKEN` | HuggingFace API token (for gated models) |
| `S3_ENDPOINT` | MinIO endpoint (local: `http://localhost:9000`) |
| `S3_BUCKET` | Default bucket name |
| `S3_ACCESS_KEY` | MinIO/S3 access key |
| `S3_SECRET_KEY` | MinIO/S3 secret key |
