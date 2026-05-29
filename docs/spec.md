# DigitalAILabIndia — Feature Specification

**Version:** 0.1.0-draft  
**Author:** Ujjwal Joshi  
**Status:** Draft — pre-implementation

---

## 1. Overview & Problem Statement

India has 22 scheduled languages and 121 million+ speakers of regional languages who are underserved by current AI tools (primarily English-optimized). Indian developers and researchers face three core problems:

1. **Data scarcity** — Curated, high-quality Indian-language datasets are scattered across academic papers, government portals, and research labs with no unified discovery layer.
2. **Fine-tuning friction** — Setting up PEFT/LoRA fine-tuning pipelines requires ML infrastructure expertise that most Indian students and small-team devs don't have.
3. **Inference access** — Running HuggingFace models for Indian languages requires GPU hardware or paid APIs with English-centric pricing.

**DigitalAILabIndia solves this** by providing a single platform: discover datasets, run inference, fine-tune models, and share results — all via a clean web UI and REST API.

---

## 2. Supported Indian Languages (v0.1 scope)

| Language | ISO Code | Script | Priority |
|---|---|---|---|
| Hindi | hi | Devanagari | P0 |
| Tamil | ta | Tamil | P0 |
| Telugu | te | Telugu | P0 |
| Bengali | bn | Bengali | P0 |
| Kannada | kn | Kannada | P1 |
| Marathi | mr | Devanagari | P1 |
| Gujarati | gu | Gujarati | P1 |
| Malayalam | ml | Malayalam | P1 |
| Punjabi | pa | Gurmukhi | P2 |
| Urdu | ur | Nastaliq | P2 |

---

## 3. Functional Requirements

### 3.1 Authentication & User Management
- [ ] User can register with email + password
- [ ] User can log in and receive a JWT access + refresh token pair
- [ ] User can store their HuggingFace API token (encrypted at rest)
- [ ] Roles: `viewer` (read-only), `contributor` (upload datasets/models), `admin` (full access)
- [ ] JWT access tokens expire in 1 hour; refresh tokens in 30 days

### 3.2 Dataset Hub
- [ ] User can browse all public datasets with pagination (20 per page)
- [ ] User can filter by: language (multi-select), domain (NLP task type), size, license
- [ ] User can search datasets by name or description (full-text search)
- [ ] Contributor can upload a dataset (CSV/JSON/JSONL, max 500MB)
- [ ] Upload validation: schema check against `data/schemas/<task_type>.json`
- [ ] Dataset metadata: name, description, language(s), domain, row count, size, license, source URL, example rows (JSON preview)
- [ ] User can download a dataset via a pre-signed S3 URL (valid 15 minutes)
- [ ] Dataset versioning: immutable versions with `v1`, `v2` tags

### 3.3 NLP Inference API
- [ ] Endpoint: `POST /inference/classify` — text classification (sentiment, topic)
- [ ] Endpoint: `POST /inference/ner` — named entity recognition
- [ ] Endpoint: `POST /inference/summarize` — extractive/abstractive summarization
- [ ] Endpoint: `POST /inference/translate` — Indian language ↔ English (v0.2)
- [ ] Endpoint: `POST /inference/transliterate` — script conversion (e.g., Hindi → Roman) (v0.2)
- [ ] Request: `{ "text": "...", "language": "hi", "model_id": "optional" }`
- [ ] Response: `{ "result": {...}, "model_used": "...", "latency_ms": 123 }`
- [ ] Default models served: `ai4bharat/indic-bert`, `google/muril-base-cased`
- [ ] Rate limiting: 10 req/min (unauthenticated), 60 req/min (authenticated)

### 3.4 Fine-Tuning Engine
- [ ] User can submit a fine-tuning job specifying: base model, dataset_id, task type, LoRA config
- [ ] LoRA config parameters: `rank` (4–64), `alpha`, `dropout`, `target_modules`, `learning_rate`, `epochs`, `batch_size`
- [ ] Job status lifecycle: `queued → running → completed | failed | cancelled`
- [ ] User can view real-time logs via polling `GET /finetune/{job_id}/logs`
- [ ] User can cancel a queued or running job
- [ ] Completed jobs auto-register the adapter in Model Registry
- [ ] Fine-tuned model is stored as a LoRA adapter (not full model) to minimize storage

### 3.5 Model Registry
- [ ] Browse all fine-tuned models with: base model, language, task, benchmark scores, created_at
- [ ] Download a model adapter (pre-signed URL)
- [ ] Use a registered model for inference by passing `model_id` to inference endpoints
- [ ] Model cards: auto-generated summary of training config + evaluation results

### 3.6 Evaluation Suite
- [ ] Run standardized benchmarks against any registered model
- [ ] Supported benchmarks v0.1: IndicGLUE (subset), language ID accuracy, NER F1
- [ ] CLI: `python scripts/run_benchmark.py --model-id <id> --benchmark indicglue`
- [ ] Store benchmark results linked to model version (immutable)

---

## 4. Non-Functional Requirements

- [ ] API p95 latency < 500ms for non-ML endpoints (auth, dataset listing)
- [ ] Inference latency p95 < 2000ms on CPU for classification (< 512 token input)
- [ ] System supports 100 concurrent users without degradation
- [ ] All API endpoints return errors in consistent format: `{ "error": "...", "code": "...", "details": {} }`
- [ ] 100% of API endpoints have OpenAPI schema documentation
- [ ] All ML models are loaded lazily and cached in memory (not reloaded per request)
- [ ] Uploaded datasets stored with server-side encryption (SSE-S3)
- [ ] Passwords hashed with bcrypt (cost factor 12)
- [ ] All secrets loaded from environment variables (never hardcoded)
- [ ] Database migrations are reversible (alembic downgrade works)

---

## 5. Data Models

### User
```
id          UUID PK
email       VARCHAR(255) UNIQUE NOT NULL
name        VARCHAR(255)
role        ENUM('viewer', 'contributor', 'admin') DEFAULT 'viewer'
hf_token    TEXT (encrypted, nullable)
created_at  TIMESTAMPTZ
updated_at  TIMESTAMPTZ
```

### Dataset
```
id              UUID PK
name            VARCHAR(255) NOT NULL
slug            VARCHAR(255) UNIQUE NOT NULL  -- url-safe
description     TEXT
language_codes  TEXT[] NOT NULL              -- e.g. ['hi', 'ta']
domain          VARCHAR(100)                 -- sentiment, ner, qa, translation, etc.
row_count       INTEGER
size_bytes      BIGINT
license         VARCHAR(100)
source_url      TEXT
s3_key          TEXT
schema_version  VARCHAR(50)
is_public       BOOLEAN DEFAULT true
version         VARCHAR(20) DEFAULT 'v1'
owner_id        UUID FK → users.id
created_at      TIMESTAMPTZ
```

### ModelVersion
```
id              UUID PK
name            VARCHAR(255)
base_model      VARCHAR(255) NOT NULL        -- HuggingFace model ID
adapter_s3_key  TEXT NOT NULL
language_codes  TEXT[]
task_type       VARCHAR(100)
config          JSONB                        -- training hyperparameters
benchmark_scores JSONB                       -- { "indicglue": 0.82, "ner_f1": 0.79 }
model_card      TEXT
is_public       BOOLEAN DEFAULT true
owner_id        UUID FK → users.id
job_id          UUID FK → fine_tuning_jobs.id (nullable)
created_at      TIMESTAMPTZ
```

### FineTuningJob
```
id              UUID PK
user_id         UUID FK → users.id
dataset_id      UUID FK → datasets.id
base_model      VARCHAR(255) NOT NULL
config          JSONB NOT NULL               -- LoRA + training config
status          ENUM('queued','running','completed','failed','cancelled')
celery_task_id  VARCHAR(255)
logs            TEXT
error_message   TEXT
started_at      TIMESTAMPTZ
finished_at     TIMESTAMPTZ
created_at      TIMESTAMPTZ
```

---

## 6. API Design

### Base URL
```
/api/v1/
```

### Auth Endpoints
| Method | Path | Auth | Description |
|---|---|---|---|
| POST | /auth/register | None | Register new user |
| POST | /auth/login | None | Login, get JWT pair |
| POST | /auth/refresh | Refresh token | Get new access token |
| GET | /auth/me | Bearer | Get current user profile |

### Dataset Endpoints
| Method | Path | Auth | Description |
|---|---|---|---|
| GET | /datasets | Optional | List datasets (paginated, filterable) |
| GET | /datasets/{id} | Optional | Get dataset metadata |
| POST | /datasets | Contributor | Upload new dataset |
| GET | /datasets/{id}/download | Bearer | Get pre-signed download URL |
| GET | /datasets/{id}/preview | Optional | Get first 10 rows as JSON |
| DELETE | /datasets/{id} | Owner/Admin | Delete dataset |

### Inference Endpoints
| Method | Path | Auth | Description |
|---|---|---|---|
| POST | /inference/classify | Optional | Text classification |
| POST | /inference/ner | Optional | Named entity recognition |
| POST | /inference/summarize | Optional | Text summarization |
| GET | /inference/models | None | List available models |

### Fine-Tuning Endpoints
| Method | Path | Auth | Description |
|---|---|---|---|
| POST | /finetune | Contributor | Submit fine-tuning job |
| GET | /finetune | Bearer | List user's jobs |
| GET | /finetune/{id} | Bearer | Get job status + config |
| GET | /finetune/{id}/logs | Bearer | Get job logs (polling) |
| DELETE | /finetune/{id} | Bearer | Cancel/delete job |

### Model Registry Endpoints
| Method | Path | Auth | Description |
|---|---|---|---|
| GET | /models | Optional | List registered models |
| GET | /models/{id} | Optional | Get model details + card |
| GET | /models/{id}/download | Bearer | Pre-signed adapter download URL |
| DELETE | /models/{id} | Owner/Admin | Delete model |

---

## 7. Test Plan

### Unit Tests
- Auth service: password hashing, JWT creation/validation, token expiry
- Dataset service: schema validation, language filter, pagination logic
- NLP service: mock HuggingFace pipeline, test input/output shapes
- Fine-tuning service: config validation, status transitions, error handling
- Model registry: benchmark score storage, model card generation

### Integration Tests
- Full auth flow: register → login → refresh → protected endpoint
- Dataset upload → storage in MinIO → download via presigned URL
- Fine-tuning job: submit → queue → poll status → model registered
- Inference endpoint with real model (use small DistilBERT for speed in CI)

### Edge Cases
- Empty text input to inference endpoints → 422
- Dataset upload exceeding 500MB → 413
- Fine-tuning with invalid LoRA config → 400 with field errors
- Concurrent fine-tuning job limit per user (max 3 queued)
- NER on text with no entities → empty list result (not 500)
- Download URL used after 15-minute expiry → 403

---

## 8. Open Questions

| # | Question | Owner | Decision |
|---|---|---|---|
| 1 | GPU availability for fine-tuning? CPU-only LoRA is slow. Consider integrating with vast.ai or Hugging Face Spaces for GPU jobs. | Ujjwal | TBD |
| 2 | Dataset licensing enforcement — how to verify contributor has rights to upload? | Ujjwal | Honor system v0.1, review workflow v0.2 |
| 3 | Should inference results be cached (Redis)? Same text + same model = same output. | Ujjwal | TBD — measure hit rate first |
| 4 | Indic script rendering in frontend — need to test cross-platform font support. | Ujjwal | Add Noto Serif fonts as fallback |
| 5 | Multi-language in a single dataset (e.g., code-switched Hindi-English) — how to tag? | Ujjwal | Allow `language_codes: ['hi', 'en']` array |
