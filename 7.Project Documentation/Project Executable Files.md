# Project Executable Files — Personalized Networking Assistant

> **Project:** Personalized Networking Assistant  
> **Repository:** [https://github.com/Sumiyazainab2308/Personalized-Networking-Assistant](https://github.com/Sumiyazainab2308/Personalized-Networking-Assistant)  
> **Stack:** FastAPI · Streamlit · BART · GPT-2 · Wikipedia API · Docker  
> **Last Updated:** July 2026

---

## Table of Contents

1. [Repository Structure](#1-repository-structure)
2. [Executable Files & Their Purpose](#2-executable-files--their-purpose)
3. [Step-by-Step Setup Instructions](#3-step-by-step-setup-instructions)
4. [Configuration Files](#4-configuration-files)
5. [Running Tests](#5-running-tests)
6. [Docker Deployment](#6-docker-deployment)
7. [Accessing the Application](#7-accessing-the-application)
8. [Troubleshooting Common Issues](#8-troubleshooting-common-issues)

---

## 1. Repository Structure

Clone URL: `https://github.com/Sumiyazainab2308/Personalized-Networking-Assistant.git`

```
Personalized-Networking-Assistant/
│
├── app/                            # FastAPI backend application
│   ├── __init__.py
│   ├── main.py                     # ★ FastAPI entry point
│   ├── config.py                   # App configuration & env loading
│   ├── models/
│   │   └── schemas.py              # Pydantic request/response models
│   ├── routers/
│   │   ├── analyze.py              # POST /analyze route
│   │   ├── generate.py             # POST /generate route
│   │   ├── factcheck.py            # POST /factcheck route
│   │   ├── history.py              # GET  /history route
│   │   └── feedback.py             # POST /feedback route
│   └── services/
│       ├── event_analyzer.py       # BART-based theme extraction
│       ├── topic_generator.py      # GPT-2-based topic generation
│       ├── fact_checker.py         # Wikipedia API fact validation
│       ├── history_manager.py      # Session history persistence
│       └── feedback_handler.py     # User feedback storage
│
├── frontend/
│   └── streamlit_app.py            # ★ Streamlit UI entry point
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py                 # Shared fixtures
│   ├── test_event_analyzer.py      # 10 unit tests
│   ├── test_topic_generator.py     # 10 unit tests
│   ├── test_fact_checker.py        # 12 unit tests
│   └── test_api_routes.py          # 6 integration tests
│
├── .env.example                    # Environment variable template
├── .env                            # ★ Your local config (gitignored)
├── .gitignore
├── docker-compose.yml              # ★ Docker orchestration
├── Dockerfile.api                  # Docker image for FastAPI
├── Dockerfile.frontend             # Docker image for Streamlit
├── Makefile                        # ★ Convenience commands
├── requirements.txt                # Python dependencies
└── README.md                       # Project overview
```

---

## 2. Executable Files & Their Purpose

### 2.1 `app/main.py` — FastAPI Entry Point

**Purpose:** The root of the FastAPI backend. It creates the `FastAPI` application instance, registers all routers, configures CORS middleware, and exposes the API health check at `GET /`.

**Key responsibilities:**
- Instantiates `FastAPI` with title, version, and description metadata
- Includes all 5 routers (`analyze`, `generate`, `factcheck`, `history`, `feedback`)
- Sets up CORS to allow the Streamlit frontend to communicate cross-origin
- Triggers model pre-loading on startup (via `@app.on_event("startup")`)

**How to run:**

```bash
# Development mode (auto-reload on file change)
uvicorn app.main:app --reload

# Specify host and port explicitly
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# Production (multiple workers, no reload)
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

**Expected output:**

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [PID] using WatchFiles
INFO:     Started server process [PID]
INFO:     Waiting for application startup.
INFO:     Loading BART model... (this may take ~60s on first run)
INFO:     Loading GPT-2 model...
INFO:     Application startup complete.
```

---

### 2.2 `frontend/streamlit_app.py` — Streamlit UI Entry Point

**Purpose:** The Streamlit-based web frontend. Provides an interactive, browser-based user interface with three tabbed pages: Event Analyzer, Topic Generator, and Fact Checker.

**Key responsibilities:**
- Renders three workflow tabs using `st.tabs()`
- Accepts user input (event name, description, industry, user profile)
- Calls the FastAPI backend via `requests` (configurable via `API_BASE_URL`)
- Displays formatted results: themes, topics, confidence scores, Wikipedia sources
- Stores session history using `st.session_state`

**How to run:**

```bash
# Default (runs on port 8501)
streamlit run frontend/streamlit_app.py

# Custom port
streamlit run frontend/streamlit_app.py --server.port 8502

# Disable automatic browser opening
streamlit run frontend/streamlit_app.py --server.headless true
```

**Expected output:**

```
  You can now view your Streamlit app in your browser.
  Local URL: http://localhost:8501
  Network URL: http://192.168.x.x:8501
```

---

### 2.3 `Makefile` — Convenience Commands

**Purpose:** Provides short, memorable `make` commands to run the API, frontend, tests, and Docker operations without remembering the full command syntax.

**Available targets:**

| Command | Equivalent | Description |
|---------|-----------|-------------|
| `make run-api` | `uvicorn app.main:app --reload` | Start FastAPI development server |
| `make run-frontend` | `streamlit run frontend/streamlit_app.py` | Start Streamlit frontend |
| `make test` | `pytest tests/ -v --cov=app --cov-report=term-missing` | Run full test suite with coverage |
| `make docker-up` | `docker compose up --build` | Build and start all Docker containers |
| `make docker-down` | `docker compose down` | Stop and remove Docker containers |
| `make install` | `pip install -r requirements.txt` | Install Python dependencies |
| `make clean` | Removes `__pycache__`, `.pytest_cache`, `htmlcov` | Clean build artifacts |
| `make lint` | `flake8 app/ frontend/ tests/` | Run code linter |

**Makefile contents (excerpt):**

```makefile
.PHONY: run-api run-frontend test docker-up docker-down install clean lint

run-api:
	uvicorn app.main:app --reload

run-frontend:
	streamlit run frontend/streamlit_app.py

test:
	pytest tests/ -v --cov=app --cov-report=term-missing

docker-up:
	docker compose up --build

docker-down:
	docker compose down

install:
	pip install -r requirements.txt

clean:
	find . -type d -name "__pycache__" -exec rm -rf {} +
	rm -rf .pytest_cache htmlcov .coverage

lint:
	flake8 app/ frontend/ tests/ --max-line-length=100
```

**How to use:**

```bash
make run-api        # Terminal 1
make run-frontend   # Terminal 2
make test           # Terminal 3
```

---

### 2.4 `docker-compose.yml` — Docker Orchestration

**Purpose:** Defines and orchestrates the multi-container Docker deployment, running the FastAPI backend and Streamlit frontend as separate isolated services that communicate over a shared Docker network.

**Services defined:**

| Service | Image | Port | Description |
|---------|-------|------|-------------|
| `api` | Built from `Dockerfile.api` | `8000:8000` | FastAPI + AI models backend |
| `frontend` | Built from `Dockerfile.frontend` | `8501:8501` | Streamlit web interface |

**`docker-compose.yml` contents:**

```yaml
version: "3.9"

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile.api
    container_name: networking-assistant-api
    ports:
      - "8000:8000"
    env_file:
      - .env
    volumes:
      - ./app:/app/app           # Live-reload in development
      - model-cache:/root/.cache  # Persist downloaded model weights
    networks:
      - assistant-net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    build:
      context: .
      dockerfile: Dockerfile.frontend
    container_name: networking-assistant-frontend
    ports:
      - "8501:8501"
    environment:
      - API_BASE_URL=http://api:8000
    depends_on:
      api:
        condition: service_healthy
    networks:
      - assistant-net

networks:
  assistant-net:
    driver: bridge

volumes:
  model-cache:
```

**How to run:**

```bash
# Build images and start all containers
docker compose up --build

# Run in detached (background) mode
docker compose up --build -d

# View container logs
docker compose logs -f

# Stop all containers
docker compose down

# Stop and remove volumes (clears model cache)
docker compose down -v
```

---

## 3. Step-by-Step Setup Instructions

Follow these steps in order to get the application running locally from scratch.

### Step 1 — Clone the Repository

```bash
git clone https://github.com/Sumiyazainab2308/Personalized-Networking-Assistant.git
cd Personalized-Networking-Assistant
```

### Step 2 — Create a Virtual Environment

Using a virtual environment isolates project dependencies from your system Python.

```bash
# Create the virtual environment
python -m venv venv

# Activate it
# On Linux / macOS:
source venv/bin/activate

# On Windows (PowerShell):
venv\Scripts\Activate.ps1

# On Windows (Command Prompt):
venv\Scripts\activate.bat
```

Verify activation — your prompt should show `(venv)`:

```
(venv) C:\...\Personalized-Networking-Assistant>
```

### Step 3 — Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

This installs: `fastapi`, `uvicorn`, `streamlit`, `transformers`, `torch`, `wikipedia-api`, `pydantic`, `requests`, `python-dotenv`, `pytest`, `pytest-cov`, and all other project dependencies.

> [!NOTE]
> Installing `torch` and `transformers` may take several minutes and download ~1–2 GB of packages depending on your platform.

### Step 4 — Configure Environment Variables

```bash
# Copy the example environment file
cp .env.example .env        # Linux/macOS
copy .env.example .env      # Windows Command Prompt
```

Open `.env` in a text editor and fill in any required values (see [Section 4](#4-configuration-files) for field descriptions).

### Step 5 — Start the FastAPI Backend

Open a terminal (or use Terminal 1):

```bash
uvicorn app.main:app --reload
```

Wait for the models to load (~60 seconds on first run). You should see:

```
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000
```

Verify the API is healthy: [http://localhost:8000/](http://localhost:8000/) — should return `{"status": "ok"}`.

### Step 6 — Start the Streamlit Frontend

Open a **second** terminal (keep the API server running in Terminal 1):

```bash
streamlit run frontend/streamlit_app.py
```

Your browser should automatically open to: [http://localhost:8501](http://localhost:8501)

If it doesn't, open it manually.

---

## 4. Configuration Files

### 4.1 `.env.example`

This file serves as the template for all required environment variables. **Never commit `.env` to Git** — it is listed in `.gitignore`.

```env
# ─── Application Settings ──────────────────────────────────────
APP_ENV=development                  # "development" | "production" | "testing"
APP_TITLE=Personalized Networking Assistant
APP_VERSION=1.0.0
LOG_LEVEL=INFO                       # "DEBUG" | "INFO" | "WARNING" | "ERROR"

# ─── API Server Settings ───────────────────────────────────────
API_HOST=0.0.0.0
API_PORT=8000
ALLOWED_ORIGINS=http://localhost:8501,http://127.0.0.1:8501

# ─── AI Model Settings ─────────────────────────────────────────
BART_MODEL_NAME=facebook/bart-large-cnn   # BART model identifier (HuggingFace)
GPT2_MODEL_NAME=gpt2                       # GPT-2 model identifier
MODEL_CACHE_DIR=./model_cache              # Directory to cache downloaded models
MAX_SUMMARY_LENGTH=150                     # Max tokens for BART summary
MAX_GENERATION_LENGTH=200                  # Max tokens for GPT-2 output
NUM_TOPICS=5                               # Number of topics to generate per request

# ─── Wikipedia API Settings ────────────────────────────────────
WIKIPEDIA_LANGUAGE=en                      # Wikipedia language code
WIKIPEDIA_USER_AGENT=PersonalizedNetworkingAssistant/1.0

# ─── Frontend Settings ─────────────────────────────────────────
API_BASE_URL=http://localhost:8000         # Streamlit → FastAPI URL

# ─── Data Storage ──────────────────────────────────────────────
HISTORY_FILE=./data/session_history.json   # Path to session history file
FEEDBACK_FILE=./data/feedback.json         # Path to feedback storage file
```

### 4.2 Environment Variable Reference

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `APP_ENV` | No | `development` | Controls debug mode and logging verbosity |
| `BART_MODEL_NAME` | No | `facebook/bart-large-cnn` | HuggingFace model ID for BART |
| `GPT2_MODEL_NAME` | No | `gpt2` | HuggingFace model ID for GPT-2 |
| `MODEL_CACHE_DIR` | No | `./model_cache` | Local path where model weights are cached |
| `API_BASE_URL` | Yes (frontend) | `http://localhost:8000` | URL that Streamlit uses to call the API |
| `ALLOWED_ORIGINS` | No | `*` | Comma-separated CORS origins |
| `WIKIPEDIA_USER_AGENT` | Yes | — | Required by Wikipedia's API terms of service |
| `HISTORY_FILE` | No | `./data/session_history.json` | JSON file for persisting history |

---

## 5. Running Tests

### Run the Full Test Suite

```bash
pytest tests/ -v
```

### Run with Coverage Report

```bash
pytest tests/ -v --cov=app --cov-report=term-missing
```

### Generate HTML Coverage Report

```bash
pytest tests/ -v --cov=app --cov-report=html
# Open htmlcov/index.html in your browser
```

### Run a Specific Test File

```bash
pytest tests/test_event_analyzer.py -v
pytest tests/test_fact_checker.py -v
```

### Run via Makefile

```bash
make test
```

Expected output:

```
========================= test session starts ==========================
collected 38 items

tests/test_event_analyzer.py::test_extract_themes_basic PASSED    [  2%]
tests/test_event_analyzer.py::test_extract_themes_returns_list PASSED [  5%]
...
tests/test_api_routes.py::test_submit_feedback PASSED             [100%]

========================= 38 passed in 3.42s ===========================

---------- coverage: platform win32, python 3.10 ----------
TOTAL    387    39    90%
```

---

## 6. Docker Deployment

### 6.1 Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- Docker Compose v2 (`docker compose` — note: no hyphen in v2)

### 6.2 Build and Start

```bash
# Build all images and start all containers
docker compose up --build
```

First build downloads model weights (~1.5 GB) into the `model-cache` Docker volume. Subsequent starts reuse the cache.

### 6.3 Running in Background

```bash
docker compose up --build -d
docker compose logs -f api        # Stream API logs
docker compose logs -f frontend   # Stream frontend logs
```

### 6.4 Stopping Containers

```bash
docker compose down               # Stop, keep volumes
docker compose down -v            # Stop and delete model cache volume
```

### 6.5 Rebuilding a Single Service

```bash
docker compose build api
docker compose up api
```

### 6.6 Dockerfile Overview

**`Dockerfile.api`:**

```dockerfile
FROM python:3.10-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app/ ./app/
COPY .env.example .env

EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**`Dockerfile.frontend`:**

```dockerfile
FROM python:3.10-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir streamlit requests python-dotenv

COPY frontend/ ./frontend/

EXPOSE 8501
CMD ["streamlit", "run", "frontend/streamlit_app.py", \
     "--server.address", "0.0.0.0", "--server.headless", "true"]
```

---

## 7. Accessing the Application

Once the API and frontend are running (locally or via Docker), use the following URLs:

| Interface | URL | Description |
|-----------|-----|-------------|
| **Streamlit UI** | [http://localhost:8501](http://localhost:8501) | Main user interface — Event Analyzer, Topic Generator, Fact Checker |
| **FastAPI Swagger UI** | [http://localhost:8000/docs](http://localhost:8000/docs) | Interactive API documentation — test all endpoints |
| **FastAPI ReDoc** | [http://localhost:8000/redoc](http://localhost:8000/redoc) | Alternative API documentation (read-only) |
| **API Health Check** | [http://localhost:8000/](http://localhost:8000/) | Returns `{"status": "ok"}` — confirm API is running |
| **OpenAPI Schema** | [http://localhost:8000/openapi.json](http://localhost:8000/openapi.json) | Raw OpenAPI 3.0 schema (JSON) |

### 7.1 Streamlit UI Pages

| Tab | Functionality |
|-----|--------------|
| 🎯 **Event Analyzer** | Enter event name + description → get themes, summary, and talking points |
| 💬 **Topic Generator** | Enter your name, role, and event themes → get personalized conversation starters |
| ✅ **Fact Checker** | Enter any claim → get Wikipedia-backed confidence score and source links |

---

## 8. Troubleshooting Common Issues

### Issue: `uvicorn: command not found`

**Cause:** Virtual environment not activated, or `uvicorn` not installed.

```bash
# Activate venv first
source venv/bin/activate       # Linux/macOS
venv\Scripts\Activate.ps1     # Windows PowerShell

# Then install
pip install uvicorn
```

---

### Issue: API server starts but models never finish loading

**Cause:** Downloading large model weights (~1.4 GB for BART) on a slow connection.

**Fix:** Be patient — first load can take 5–15 minutes depending on internet speed. Subsequent starts will use the local cache and take ~60 seconds (CPU load).

```bash
# Check if models are downloading
# Watch the logs for: "Downloading model weights..."
```

---

### Issue: Streamlit shows `Connection refused` when calling the API

**Cause:** `API_BASE_URL` in `.env` does not match the actual API host/port.

**Fix:**

```env
# In .env (for local development)
API_BASE_URL=http://localhost:8000

# In .env (when using Docker Compose)
API_BASE_URL=http://api:8000
```

---

### Issue: `CORS error` in browser console

**Cause:** The FastAPI `ALLOWED_ORIGINS` does not include the Streamlit URL.

**Fix:**

```env
# In .env
ALLOWED_ORIGINS=http://localhost:8501,http://127.0.0.1:8501
```

---

### Issue: `ModuleNotFoundError: No module named 'app'`

**Cause:** Tests or the server are run from the wrong directory.

**Fix:** Always run commands from the **project root** (the directory containing `app/`, `frontend/`, and `requirements.txt`):

```bash
cd Personalized-Networking-Assistant
pytest tests/ -v                     # ✅ Correct
cd tests && pytest .                 # ❌ Wrong — imports will fail
```

---

### Issue: Docker build fails with `no space left on device`

**Cause:** Docker's disk usage limit is exceeded (model weights are large).

**Fix:**

```bash
# Clean up unused Docker resources
docker system prune -af
docker volume prune -f
```

---

### Issue: `Port 8000 is already in use`

**Cause:** Another process is using port 8000.

```bash
# Find and kill the process (Linux/macOS)
lsof -i :8000
kill -9 <PID>

# Windows PowerShell
netstat -ano | findstr :8000
taskkill /PID <PID> /F

# Or run on a different port
uvicorn app.main:app --port 8001
```

---

*Document prepared by the Personalized Networking Assistant development team.*  
*Team Lead: Shaik Sumiya Zainab | Members: Naga Jagan Mohan Rao Thattukolla, Satvika Tallam, Tejesh Velivela*
