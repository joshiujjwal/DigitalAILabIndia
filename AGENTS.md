# AGENTS.md — DigitalAILabIndia

Coding agent instructions. Follow these exactly.

---

## Setup

```bash
# 1. Python environment
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt -r requirements-dev.txt

# 2. Infrastructure
docker compose up -d db redis minio

# 3. Database
cp .env.example .env        # fill in values
alembic upgrade head

# 4. Start backend
uvicorn src.api.main:app --reload --port 8000

# 5. Frontend
cd frontend && npm install && npm run dev
```

---

## Testing

**Run tests before writing any code. All tests must pass before you commit.**

```bash
# All backend tests
pytest tests/ -v --cov=src --cov-report=term-missing

# Specific module
pytest tests/unit/test_auth.py -v

# Frontend tests
cd frontend && npm run test

# Type checks
mypy src/
cd frontend && npm run typecheck
```

**TDD is mandatory:**
1. Write the failing test → confirm RED
2. Write minimum implementation → confirm GREEN
3. Refactor only after GREEN

---

## Code Style

### Python
- Python 3.11+ — use `match/case`, `X | Y` union types, `TypeAlias`
- Ruff for linting and formatting: `ruff check src/ && ruff format src/`
- Type annotations on every function signature — no bare `def foo(x):`
- Pydantic v2 for all request/response schemas — use `model_validate`, `model_dump`
- SQLAlchemy 2.0 style — use `select()`, `session.execute()`, not legacy `session.query()`
- Async/await throughout — all DB calls must be non-blocking
- Exception hierarchy: define custom exceptions in `src/utils/exceptions.py`
- Max line length: 100 chars (configured in pyproject.toml)

### TypeScript / React
- TypeScript strict mode — zero `any`, zero `@ts-ignore`
- Functional components only — no class components
- `const` over `let`; `let` over `var` (never `var`)
- Named exports only — no default exports from component files (except pages)
- All API interfaces defined in `frontend/src/types/index.ts`
- TailwindCSS only for styling — no inline styles, no CSS modules
- Error boundaries on async data-heavy pages

---

## PR Instructions

Every PR must include:

1. **Test evidence** — paste output of `pytest tests/ -v` (or subset relevant to PR)
2. **What changed** — bullet list of functional changes (not a git log)
3. **What was NOT changed** — explicitly state scope boundaries
4. **For ML changes** — include before/after benchmark numbers if modifying inference/fine-tuning

**Do not:**
- Remove or skip existing tests without explicit approval
- Add `type: ignore` comments without an explanatory comment
- Refactor code outside the scope of your assigned task
- Hardcode environment-specific values (IPs, paths, tokens)
- Merge without CI passing

---

## Architecture Rules

1. **Routers → Services → Models** — strict one-way dependency. Routers call services, services call ORM models. Never reverse this.
2. **All secrets via env vars** — use `src/utils/config.py` (Pydantic Settings). Never `os.environ["KEY"]` directly in business logic.
3. **Celery tasks are thin** — they call service functions, they don't contain business logic themselves.
4. **S3 paths** follow pattern: `{entity_type}/{owner_id}/{uuid}/{filename}` (e.g., `datasets/user-uuid/dataset-uuid/data.jsonl`)
5. **All timestamps are UTC** — use `datetime.now(timezone.utc)`. Never naive datetimes.
6. **Language codes are validated** against the supported list in `src/utils/model_catalog.py` at the API boundary.

---

## Benchmark Models for Tests

Use these small, fast models in tests (not full-size production models):

| Task | Test Model |
|---|---|
| Classification | `distilbert-base-uncased` |
| NER | `dslim/bert-base-NER` |
| Summarization | `sshleifer/distilbart-cnn-6-6` |

Never load `ai4bharat/indic-bert` or `google/muril-base-cased` in unit tests — they are too large. Mock the HF pipeline instead.
