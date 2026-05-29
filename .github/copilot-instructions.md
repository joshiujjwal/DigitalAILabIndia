# GitHub Copilot Instructions — DigitalAILabIndia

## Project Overview
AI research and experimentation platform for Indian languages and use cases. Backend: Python 3.11 + FastAPI + SQLAlchemy 2.0 + Celery. ML: HuggingFace Transformers + PEFT/LoRA. Database: PostgreSQL 15. Frontend: React 18 + TypeScript + Vite + TailwindCSS.

---

## Stack & Conventions

### Backend (Python)
- All route handlers are `async def`
- Use Pydantic v2 — `model_validate()`, `model_dump()`, `Field()` with descriptions
- SQLAlchemy 2.0 style: `select(Model).where(...)`, `await session.execute(...)`
- UUIDs for all primary keys: `id: Mapped[UUID] = mapped_column(default=uuid4)`
- Timestamps always UTC: `datetime.now(timezone.utc)`
- Custom exceptions in `src/utils/exceptions.py`; caught and converted in routers
- Routers only call service functions — no ORM calls in router files
- Config via Pydantic Settings in `src/utils/config.py` — access as `settings.DATABASE_URL`

### Frontend (TypeScript / React)
- Functional components with hooks only
- All types in `frontend/src/types/index.ts` — import from there, never re-declare
- All API calls through `frontend/src/services/api.ts` — never raw `fetch`
- TailwindCSS for all styling — no CSS-in-JS, no modules
- Use `Noto Sans` / `Noto Serif Devanagari` font stack for text areas displaying Indic languages
- React Query for server state (data fetching, caching, mutation)
- Zod for form validation on fine-tuning config forms

### Indian Language Handling
- Language codes: ISO 639-1 (`hi`, `ta`, `te`, `bn`, `kn`, `mr`, `gu`, `ml`, `pa`, `ur`)
- Normalize Indic text: `unicodedata.normalize('NFC', text)` before ML inference
- RTL support: Urdu (`ur`) requires `dir="rtl"` on text display elements
- Never assume ASCII — all text fields are Unicode (Postgres `TEXT`, Python `str`)

---

## Testing Conventions

- **Write failing tests FIRST** before any implementation
- Backend: `pytest` + `pytest-asyncio` + `factory_boy` for fixtures + `pytest-mock` for mocks
- Frontend: Vitest + React Testing Library
- Mock HuggingFace pipelines in unit tests — never load real models in unit tests
- Use `distilbert-base-uncased` (not `indic-bert`) for integration tests that need a real model
- Test file mirrors source: `src/services/auth.py` → `tests/unit/test_auth.py`
- Integration tests use a separate test DB (`TEST_DATABASE_URL` env var)

---

## Boundaries & Rules

- **Do NOT** refactor code outside the scope of the current task
- **Do NOT** remove or comment out existing tests
- **Do NOT** add `# type: ignore` without a comment explaining why
- **Do NOT** call HuggingFace APIs in unit tests — always mock
- **Do NOT** hardcode ports, hostnames, or secrets — use `settings.*`
- **Do NOT** use synchronous SQLAlchemy in async routes — always `await`
- **Do NOT** import directly from `celery` in routers — use service functions that enqueue tasks
- **ALWAYS** validate `language_codes` against `SUPPORTED_LANGUAGES` in `src/utils/model_catalog.py`
- **ALWAYS** use UTC for any datetime comparison or storage
- **ALWAYS** return consistent error format: `{"error": "...", "code": "...", "details": {}}`
