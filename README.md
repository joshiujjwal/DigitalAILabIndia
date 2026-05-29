# DigitalAILabIndia 🇮🇳

> **Digital AI Lab for India** — An AI research and experimentation platform tailored for India.

![Status](https://img.shields.io/badge/status-🚧%20Early%20Development-orange)
![Stack](https://img.shields.io/badge/stack-Python%20%7C%20FastAPI%20%7C%20React%20%7C%20PostgreSQL-blue)
![License](https://img.shields.io/badge/license-MIT-green)

Democratizes AI access for Indian developers, researchers, and students. Provides accessible AI tools, curated Indian-language datasets, and model fine-tuning capabilities for Indian languages and use cases — from Hindi NLP to regional language ASR.

---

## ✨ Features (Planned)

| Feature | Description | Status |
|---|---|---|
| Indian Language NLP | Text classification, NER, summarization for Hindi, Tamil, Telugu, Bengali + more | 🚧 |
| Dataset Hub | Curated Indian-language datasets with metadata and download APIs | 🚧 |
| Fine-Tuning Playground | No-code/low-code interface to fine-tune HuggingFace models on custom datasets | 🚧 |
| Model Registry | Version-controlled store for fine-tuned models with benchmark scores | 🚧 |
| Evaluation Suite | Standardized benchmarks for Indian language understanding tasks | 🚧 |
| API Gateway | REST API for all AI capabilities — integrate in your apps | 🚧 |
| Community Contributions | Open dataset and model contribution pipeline with review workflow | 🚧 |

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Backend API | Python 3.11+, FastAPI, Pydantic v2 |
| ML / AI | HuggingFace Transformers, Datasets, PEFT, Accelerate |
| Database | PostgreSQL 15 (via SQLAlchemy + Alembic) |
| Task Queue | Celery + Redis |
| Frontend | React 18, TypeScript, Vite, TailwindCSS |
| Auth | JWT (FastAPI Users) |
| Storage | S3-compatible (MinIO for local dev) |
| Containerization | Docker + Docker Compose |
| CI/CD | GitHub Actions |
| Monitoring | Prometheus + Grafana |

---

## 🚀 Getting Started

### Prerequisites

- Python 3.11+
- Node.js 20+
- Docker + Docker Compose
- PostgreSQL 15 (or use Docker)
- Git

### Installation

```bash
# Clone the repo
git clone https://github.com/joshiujjwal/DigitalAILabIndia.git
cd DigitalAILabIndia

# Backend setup
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
pip install -r requirements-dev.txt

# Frontend setup
cd frontend
npm install
cd ..

# Copy and configure environment
cp .env.example .env
# Edit .env with your database URL, secret keys, HuggingFace token, etc.

# Start infrastructure (PostgreSQL, Redis, MinIO)
docker compose up -d db redis minio

# Run database migrations
alembic upgrade head

# Start the backend (dev)
uvicorn src.api.main:app --reload --port 8000

# Start the frontend (new terminal)
cd frontend && npm run dev
```

### Running Tests

```bash
# Backend tests
pytest tests/ -v --cov=src --cov-report=term-missing

# Frontend tests
cd frontend && npm run test

# E2E tests
pytest tests/e2e/ -v
```

### Linting & Formatting

```bash
# Backend
ruff check src/ tests/
ruff format src/ tests/
mypy src/

# Frontend
cd frontend && npm run lint
cd frontend && npm run typecheck
```

---

## 📂 Project Structure

```
DigitalAILabIndia/
├── src/
│   ├── api/          # FastAPI routers, dependencies, middleware
│   ├── models/       # SQLAlchemy ORM models
│   ├── services/     # Business logic (NLP, fine-tuning, dataset mgmt)
│   ├── workers/      # Celery tasks (async fine-tuning jobs)
│   └── utils/        # Shared helpers, constants, config
├── tests/
│   ├── unit/         # Unit tests mirroring src/
│   ├── integration/  # Database + API integration tests
│   └── e2e/          # End-to-end API tests
├── frontend/
│   └── src/
│       ├── components/  # Reusable UI components
│       ├── pages/       # Route-level page components
│       ├── hooks/       # Custom React hooks
│       ├── services/    # API client functions
│       └── types/       # TypeScript type definitions
├── docs/
│   ├── spec.md          # Feature specification
│   └── adr/             # Architecture Decision Records
├── scripts/             # Dev scripts, seed data, migration helpers
├── data/
│   ├── samples/         # Sample datasets for testing
│   └── schemas/         # JSON Schema for dataset validation
├── .github/
│   ├── copilot-instructions.md
│   ├── instructions/
│   └── skills/
├── README.md
├── TODO.md
├── CLAUDE.md
├── AGENTS.md
├── docker-compose.yml
└── .gitignore
```

---

## 🤝 Contributing

1. **Read `TODO.md`** — find a task in the current phase
2. **Write tests first** (red phase) before any implementation
3. **Implement** until tests pass (green phase)
4. **Review your diff** — reject anything you wouldn't approve in code review
5. **Commit** with a descriptive message referencing the TODO item
6. **Open a PR** with evidence: test output, benchmarks, or screenshots
7. PRs without passing tests or evidence will not be merged

---

## 📜 License

MIT © Ujjwal Joshi
