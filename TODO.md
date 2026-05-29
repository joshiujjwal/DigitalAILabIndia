# DigitalAILabIndia — Task Breakdown

## How to Use This File

**Every task follows this exact loop:**
1. ✍️ Write failing tests first (`pytest` should report RED)
2. 🔨 Implement the minimum code to make tests pass (GREEN)
3. 🔍 Review your own diff — reject anything you wouldn't approve in a PR
4. 💾 Commit with a message like `feat(dataset-hub): add language filter endpoint`
5. 📝 Update `CLAUDE.md` or `AGENTS.md` if you discovered a new convention or gotcha (compound loop)
6. ✅ Check off the task below

**Evidence gates** — each phase is locked until the previous phase has:
- All tests passing (`pytest` + `npm run test`)
- Human diff review complete
- Phase summary added to **Lessons Learned** section below

---

## Phase 0: Foundation ⬜

> Goal: Repo is runnable, testable, and CI is green.

- [ ] Create `pyproject.toml` with ruff, mypy, pytest config
- [ ] Create `requirements.txt` and `requirements-dev.txt` (FastAPI, SQLAlchemy, Alembic, HuggingFace, Celery, Redis, pytest, httpx, factory_boy)
- [ ] Set up `frontend/package.json` with React 18, TypeScript, Vite, TailwindCSS, Vitest, ESLint
- [ ] Create `.env.example` with all required env vars (DATABASE_URL, REDIS_URL, HF_TOKEN, SECRET_KEY, S3_ENDPOINT, S3_BUCKET)
- [ ] Create `docker-compose.yml` with services: `db` (postgres:15), `redis`, `minio`, `backend`, `frontend`, `worker`
- [ ] Write smoke test: `tests/unit/test_smoke.py` — asserts app can be imported without errors
- [ ] Write smoke test: `frontend/src/__tests__/smoke.test.tsx` — asserts App renders
- [ ] Set up `alembic init` and initial migration scaffold
- [ ] Create `.github/workflows/ci.yml` — lint, typecheck, test on every push/PR
- [ ] Verify `pytest tests/unit/test_smoke.py` passes ✅ (EVIDENCE GATE)
- [ ] Verify `npm run test` passes ✅ (EVIDENCE GATE)
- [ ] Verify `ruff check src/` returns 0 errors ✅ (EVIDENCE GATE)

**Phase 0 Review checkpoint:** CI green, docker compose up starts all services.

---

## Phase 1: Data Layer — Models & Migrations ⬜

> Goal: Database schema is defined, migrated, and tested.

- [ ] Write tests: `tests/unit/test_models.py` — test User, Dataset, Model, FineTuningJob ORM models (red)
- [ ] Implement `src/models/user.py` — User model (id, email, name, role, hf_token_enc, created_at)
- [ ] Implement `src/models/dataset.py` — Dataset model (id, name, language, domain, size_bytes, source_url, schema_version, is_public, owner_id)
- [ ] Implement `src/models/model_registry.py` — ModelVersion model (id, base_model, adapter_path, language, task_type, benchmark_scores JSON, created_at)
- [ ] Implement `src/models/fine_tuning_job.py` — FineTuningJob model (id, user_id, dataset_id, base_model, config JSON, status, logs, started_at, finished_at)
- [ ] Write + run Alembic migration: `alembic revision --autogenerate -m "initial_schema"`
- [ ] Verify `alembic upgrade head` succeeds against live Postgres ✅
- [ ] Write `tests/integration/test_db_models.py` — CRUD round-trip for each model
- [ ] Verify all model tests pass ✅ (EVIDENCE GATE)

---

## Phase 2: Authentication & Core API ⬜

> Goal: Users can register, login, and get a JWT. API skeleton is live.

- [ ] Write tests: `tests/unit/test_auth.py` — register, login, token refresh, invalid credentials (red)
- [ ] Implement `src/services/auth.py` — JWT creation, password hashing (bcrypt), token refresh
- [ ] Implement `src/api/routers/auth.py` — POST /auth/register, POST /auth/login, POST /auth/refresh
- [ ] Implement `src/api/dependencies.py` — `get_current_user` dependency, role guards
- [ ] Implement `src/api/main.py` — FastAPI app factory, CORS, middleware, router mounting
- [ ] Write `tests/integration/test_auth_api.py` — full register→login→token flow via httpx TestClient
- [ ] Verify auth tests pass ✅ (EVIDENCE GATE)

---

## Phase 3: Dataset Hub ⬜

> Goal: Users can browse, upload, download, and search Indian-language datasets.

- [ ] Write tests: `tests/unit/test_dataset_service.py` — list, filter by language, upload validation, schema check (red)
- [ ] Implement `src/services/dataset_service.py` — list_datasets, get_dataset, upload_dataset, validate_schema, generate_presigned_url
- [ ] Implement `src/api/routers/datasets.py` — GET /datasets, GET /datasets/{id}, POST /datasets, GET /datasets/{id}/download
- [ ] Implement `src/utils/s3.py` — MinIO/S3 client wrapper with presigned URL generation
- [ ] Implement `src/utils/schema_validator.py` — validate uploaded datasets against JSON schemas in `data/schemas/`
- [ ] Add sample datasets in `data/samples/` — Hindi sentiment (100 rows), Tamil NER (50 rows)
- [ ] Write `tests/integration/test_dataset_api.py` — upload, list, filter, download round-trip
- [ ] Verify all dataset tests pass ✅ (EVIDENCE GATE)

---

## Phase 4: NLP Inference API ⬜

> Goal: Users can run inference on pre-loaded Indian language models via REST.

- [ ] Write tests: `tests/unit/test_nlp_service.py` — text classification, NER, summarization (mock HF calls) (red)
- [ ] Implement `src/services/nlp_service.py` — load_model, predict_classification, predict_ner, summarize using HuggingFace pipelines
- [ ] Implement `src/api/routers/inference.py` — POST /inference/classify, POST /inference/ner, POST /inference/summarize
- [ ] Implement model lazy-loading with caching (avoid re-loading on every request)
- [ ] Add supported models config: `src/utils/model_catalog.py` (ai4bharat/indic-bert, google/muril-base-cased, etc.)
- [ ] Implement rate limiting middleware (10 req/min free tier, 100 req/min authenticated)
- [ ] Write `tests/integration/test_inference_api.py` — classify Hindi text, NER Tamil text
- [ ] Benchmark: p95 latency < 2s for classification on CPU ✅ (EVIDENCE GATE)
- [ ] Verify inference tests pass ✅ (EVIDENCE GATE)

---

## Phase 5: Fine-Tuning Engine ⬜

> Goal: Users can submit fine-tuning jobs that run asynchronously via Celery.

- [ ] Write tests: `tests/unit/test_finetuning_service.py` — job creation, config validation, status transitions (red)
- [ ] Implement `src/services/finetuning_service.py` — create_job, get_job_status, cancel_job, get_job_logs
- [ ] Implement `src/workers/finetuning_worker.py` — Celery task that runs PEFT/LoRA fine-tuning using HuggingFace Trainer
- [ ] Implement `src/api/routers/finetuning.py` — POST /finetune, GET /finetune/{job_id}, DELETE /finetune/{job_id}
- [ ] Add job config schemas: LoRA rank, learning rate, epochs, batch size, language, task type
- [ ] Implement model saving to MinIO after successful fine-tune
- [ ] Implement model registration to ModelRegistry on job completion
- [ ] Write `tests/integration/test_finetuning_api.py` — submit job, poll status, retrieve model
- [ ] Verify finetuning tests pass ✅ (EVIDENCE GATE)

---

## Phase 6: Frontend — Core UI ⬜

> Goal: React app has working Dataset Hub, Inference Playground, and Fine-Tuning dashboard.

- [ ] Write component tests: `DatasetCard`, `LanguageFilter`, `InferencePlayground`, `JobStatusTable` (red with Vitest + Testing Library)
- [ ] Implement `frontend/src/services/api.ts` — typed Axios client wrapping all backend endpoints
- [ ] Implement `frontend/src/pages/DatasetHub.tsx` — browse, search, filter datasets by language/domain
- [ ] Implement `frontend/src/pages/InferencePlayground.tsx` — model selector, text input, live results
- [ ] Implement `frontend/src/pages/FineTuningDashboard.tsx` — submit job form, job list, status polling
- [ ] Implement `frontend/src/pages/ModelRegistry.tsx` — browse fine-tuned models with benchmark scores
- [ ] Implement auth flow: Login/Register pages, JWT storage, protected routes
- [ ] Add responsive layout with TailwindCSS — works on mobile (important for Indian users)
- [ ] Verify component tests pass ✅ (EVIDENCE GATE)

---

## Phase 7: Evaluation Suite ⬜

> Goal: Standardized benchmarks for Indian language models are runnable via CLI and API.

- [ ] Define benchmark tasks: IndicGLUE tasks subset, language identification, transliteration accuracy
- [ ] Implement `scripts/run_benchmark.py` — CLI to evaluate any registered model against benchmark suite
- [ ] Implement `src/api/routers/evaluation.py` — POST /evaluate (async), GET /evaluate/{job_id}
- [ ] Implement `src/workers/eval_worker.py` — Celery task for benchmark evaluation
- [ ] Store and expose benchmark results per model version in ModelRegistry
- [ ] Verify benchmark script produces reproducible results on sample data ✅ (EVIDENCE GATE)

---

## Phase 8: Polish & Harden ⬜

> Goal: Production-ready — security, observability, performance, and documentation.

- [ ] Security: Add input sanitization, SQL injection guards, file type validation on uploads
- [ ] Security: Encrypt HuggingFace tokens at rest (Fernet/AES)
- [ ] Observability: Add Prometheus metrics endpoint (`/metrics`) — request count, latency histograms, job queue depth
- [ ] Add structured JSON logging (structlog) with trace IDs
- [ ] API docs: ensure `/docs` (Swagger) and `/redoc` are complete with examples
- [ ] Add `scripts/seed_db.py` — seed dev DB with sample users, datasets, and models
- [ ] Load testing: run `locust` against inference API — target 50 concurrent users
- [ ] Write ADR: `docs/adr/0002-peft-lora-for-finetuning.md`
- [ ] Write ADR: `docs/adr/0003-celery-vs-ray-for-async-jobs.md`
- [ ] Final pass: zero ruff/mypy errors, zero ESLint errors ✅ (EVIDENCE GATE)

---

## Phase 9: Ship 🚀 ⬜

- [ ] Create `Dockerfile` (multi-stage: builder + slim runtime) for backend
- [ ] Create `frontend/Dockerfile`
- [ ] Configure GitHub Actions: `deploy.yml` — build images, push to GHCR, deploy to target infra
- [ ] Write production `docker-compose.prod.yml` with resource limits
- [ ] Create `docs/deployment.md` — step-by-step production deployment guide
- [ ] Create `docs/api.md` — API reference with curl examples
- [ ] Tag `v0.1.0` release
- [ ] Announce on IndiaAI communities 🎉

---

## Parking Lot 🅿️

> Ideas that don't fit current phases — revisit later.

- Voice/ASR support for Indian regional languages (Whisper fine-tuning)
- Indic script OCR integration
- Community voting on dataset quality
- Federated learning for privacy-preserving fine-tuning
- Mobile SDK (React Native)
- Integration with NPCI / DigiLocker data for civic AI use cases
- GPU cluster integration (AWS/GCP spot instances)
- Hindi/regional language UI localization

---

## Lessons Learned 📝

> Updated as the project grows. Each lesson should be actionable.

_None yet — add entries as you discover non-obvious things about this codebase._

Format: `[DATE] [AREA] — What you learned and why it matters for this project.`
