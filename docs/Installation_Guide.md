# Installation, Setup, and Troubleshooting Guide
**Project:** Personalized Networking Assistant  
**Repository:** `networking-assistant`  
**Document Version:** 1.0.0  
**Date:** July 2026  
**Authors:** Shaik Sumiya Zainab, Naga Jagan Mohan Rao Thattukolla, Satvika Tallam, Tejesh Velivela  

---

## 1. Executive Summary

This document provides an exhaustive, step-by-step installation, configuration, deployment, and troubleshooting manual for the **Personalized Networking Assistant**. Designed for academic and industry GenAI/ML competition evaluators, DevOps engineers, and developers, this guide ensures clean, reproducible deployments across Windows, Linux, macOS, and Docker container environments without dependency conflicts or environmental discrepancies.

---

## 2. Prerequisites and Hardware Requirements

Before cloning the repository and initiating installation, ensure that your deployment environment satisfies the following baseline system requirements:

| System Component | Minimum Requirement | Recommended Specification | Architectural & Implementation Notes |
| :--- | :--- | :--- | :--- |
| **Operating System** | Windows 10/11 (64-bit), Ubuntu 20.04+, macOS 12+ | Windows 11 Enterprise / Ubuntu 22.04 LTS | Verified across Windows (PowerShell/WSL2), Linux, and macOS environments. |
| **Python Runtime** | CPython 3.11.0+ | CPython 3.11.8 | **Strict Requirement:** Python 3.11+ is required to leverage speed optimizations and type hinting enhancements. |
| **System Memory (RAM)** | 8 GB RAM | 16 GB DDR4/DDR5 RAM | **Critical:** Loading HuggingFace `facebook/bart-large-mnli` (~1.6GB) and `gpt2` (~500MB) into memory alongside PyTorch requires at least 8GB RAM to prevent Out-Of-Memory (OOM) errors. |
| **CPU Architecture** | 4 Cores, x86_64 or ARM64 (Apple Silicon) | 8 Cores, Intel Core i7 / AMD Ryzen 7 | 100% CPU-bound inference; no NVIDIA GPU or CUDA drivers required. |
| **Disk Storage** | 5 GB Free Disk Space | 10 GB NVMe SSD Storage | Accommodates virtual environment packages, PyTorch runtime, downloaded HuggingFace model cache (~2.5GB), and local JSON history logs. |
| **Version Control** | Git 2.30+ | Git 2.40+ | Required for cloning repository and managing branch checkouts. |
| **Container Engine** | Docker Engine 20.10+ (Optional) | Docker Desktop 4.20+ with Docker Compose v2 | Required *only* if deploying via automated Docker container orchestration. |

---

## 3. Step-by-Step Installation Guide

### Step 1: Clone the Repository
Open your command prompt, PowerShell, or terminal, and clone the official project repository to your local workspace:

```bash
# Clone the repository via HTTPS
git clone https://github.com/networking-assistant/networking-assistant.git

# Navigate into the project root directory
cd networking-assistant
```

### Step 2: Create and Activate a Python Virtual Environment
To isolate project dependencies (PyTorch, Transformers, FastAPI, Streamlit) from your system-wide Python installation, creating a virtual environment is mandatory.

#### On Windows (PowerShell or Command Prompt):
```powershell
# Create virtual environment named 'venv' using Python 3.11
python -m venv venv

# Activate virtual environment in Command Prompt (CMD):
venv\Scripts\activate

# OR Activate virtual environment in PowerShell:
.\venv\Scripts\Activate.ps1
```
*(Note: If PowerShell returns an execution policy error, run `Set-ExecutionPolicy Unrestricted -Scope Process` first).*

#### On Linux or macOS (Bash / Zsh):
```bash
# Create virtual environment named 'venv' using Python 3.11
python3 -m venv venv

# Activate virtual environment:
source venv/bin/activate
```

When successfully activated, your terminal prompt will be prefixed with `(venv)`.

### Step 3: Upgrade Pip and Install Dependencies
With the virtual environment active, upgrade Python's package manager and install the required dependencies:

```bash
# Upgrade pip to the latest version to avoid build wheel errors
python -m pip install --upgrade pip

# Install project dependencies from requirements.txt
pip install -r requirements.txt
```
*(Note: Installing `torch`, `transformers`, and `streamlit` may take 3–5 minutes depending on your internet bandwidth).*

---

## 4. Setting Up Environment Variables

The backend application utilizes `pydantic-settings` (`app/config.py`) to manage configuration parameters cleanly via environment variables.

### Step 1: Create the Environment Configuration File
Copy the provided template file `.env.example` to create your active `.env` configuration file in the root directory:

#### On Windows (Command Prompt / PowerShell):
```powershell
copy .env.example .env
```

#### On Linux / macOS (Bash / Zsh):
```bash
cp .env.example .env
```

### Step 2: Configure Environment Variables
Open the `.env` file in your preferred text editor (VS Code, Notepad, Vim) and review/customize the following parameters:

| Environment Variable | Default Value in `.env.example` | Data Type & Validation Rule | Architectural Description & Purpose |
| :--- | :--- | :--- | :--- |
| `APP_NAME` | `Personalized Networking Assistant` | String | Display name of the backend service utilized in OpenAPI documentation and logs. |
| `APP_ENV` | `development` | String (`development`, `staging`, `production`) | Application operating environment. Controls debug messaging and traceback verbosity. |
| `PORT` | `8000` | Integer (1024–65535) | The local HTTP port on which the Uvicorn FastAPI ASGI server will bind and listen. |
| `HOST` | `0.0.0.0` | String (Valid IPv4 address) | Network interface binding. Use `0.0.0.0` for Docker/LAN access or `127.0.0.1` for local-only loopback. |
| `MODEL_CACHE_DIR` | `./models_cache` | String (Directory Path) | Local filesystem directory where HuggingFace transformer weights (`BART` and `GPT-2`) are downloaded and cached. |
| `LOG_LEVEL` | `INFO` | String (`DEBUG`, `INFO`, `WARNING`, `ERROR`) | Console logging verbosity level for application events, routing requests, and AI inference timing. |
| `WIKIPEDIA_USER_AGENT` | `NetworkingAssistant/1.0 (mailto:contact@networkingassistant.com)` | String (Valid User-Agent format) | **Critical Required Header:** Custom HTTP header required by Wikipedia REST API. Without a valid User-Agent, Wikipedia API will reject requests with `HTTP 403 Forbidden`. |
| `HISTORY_FILE_PATH` | `data/history.json` | String (File Path) | Relative or absolute filesystem path where historical networking sessions are atomically written and stored. |
| `FEEDBACK_FILE_PATH` | `data/feedback.json` | String (File Path) | Relative or absolute filesystem path where user ratings and thumbs-up/down feedback logs are atomically written. |

```ini
# Example .env configuration file
APP_NAME="Personalized Networking Assistant"
APP_ENV="development"
PORT=8000
HOST="0.0.0.0"
MODEL_CACHE_DIR="./models_cache"
LOG_LEVEL="INFO"
WIKIPEDIA_USER_AGENT="NetworkingAssistant/1.0 (mailto:competition-judge@networkingassistant.com)"
HISTORY_FILE_PATH="data/history.json"
FEEDBACK_FILE_PATH="data/feedback.json"
```

---

## 5. Running the Application Locally (Manual Execution)

The architecture consists of decoupled backend and frontend services. To run the application locally without Docker, open **two separate terminal windows**, ensure `venv` is activated in both, and execute the following commands:

### 5.1 Running the FastAPI Backend Server (Port 8000)
In your first terminal window, start the FastAPI ASGI server using Uvicorn:

#### Using Direct Uvicorn Command:
```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```
*(The `--reload` flag enables auto-reloading upon file modifications during development).*

#### Using Makefile Automation (Recommended):
```bash
make run-api
```

**Verifying Backend Health & Documentation:**
Once initialized, you will see log output confirming model loading and server startup. You can verify backend connectivity and explore interactive API documentation by opening your web browser to:
- **Interactive OpenAPI Swagger UI:** [http://localhost:8000/docs](http://localhost:8000/docs)
- **Alternative ReDoc API Documentation:** [http://localhost:8000/redoc](http://localhost:8000/redoc)
- **Direct Health Check Endpoint:** [http://localhost:8000/](http://localhost:8000/)

### 5.2 Running the Streamlit Frontend UI (Port 8501)
In your second terminal window, launch the multi-page Streamlit web interface:

#### Using Direct Streamlit Command:
```bash
streamlit run frontend/streamlit_app.py --server.port 8501 --server.address 0.0.0.0
```

#### Using Makefile Automation (Recommended):
```bash
make run-frontend
```

**Accessing the User Interface:**
Upon initialization, Streamlit will automatically launch your default web browser, or you can manually navigate to:
- **Streamlit Web Application:** [http://localhost:8501](http://localhost:8501)

---

## 6. Running with Docker & Docker Compose (Containerized Deployment)

For seamless, single-command deployment across heterogeneous operating systems without manual virtual environment setup, the repository includes a fully orchestrated multi-container Docker implementation.

### 6.1 Architecture Overview
The `docker-compose.yml` file orchestrates two containerized services communicating over an isolated internal Docker network:
1. `api` **Service:** Builds from `Dockerfile`, running the FastAPI server on port `8000`. Mounts the `./data` volume to ensure atomic JSON history and feedback logs persist across container restarts. Mounts `./models_cache` to prevent re-downloading HuggingFace weights on every build.
2. `frontend` **Service:** Builds from `Dockerfile.frontend`, running the Streamlit web application on port `8501`. Communicates internally with the backend via `http://api:8000`.

### 6.2 Starting Container Services
Open your terminal in the project root directory and execute:

#### Using Docker Compose Command:
```bash
# Build Docker images and start containers in detached mode (-d)
docker compose up --build -d
```

#### Using Makefile Automation:
```bash
make docker-up
```

### 6.3 Checking Container Logs and Status
To monitor real-time container startup logs, model caching progress, and operational status:

```bash
# View live streaming logs from all running containers
docker compose logs -f

# View status and port mappings of active containers
docker compose ps
```

### 6.4 Stopping and Cleaning Up Containers
To gracefully terminate containers, remove bridge networks, and clean up temporary container artifacts (while preserving your persistent `./data` JSON logs):

#### Using Docker Compose Command:
```bash
docker compose down
```

#### Using Makefile Automation:
```bash
make docker-down
```

---

## 7. Running the Automated Test Suite & Coverage Analysis

The repository includes a comprehensive 38-test automated Quality Assurance suite (`tests/`) designed by QA Engineer Satvika Tallam. The test suite utilizes `pytest` and `unittest.mock` to perform exhaustive unit and integration testing without requiring an internet connection or GPU hardware.

### 7.1 Executing the Test Suite
Ensure your virtual environment is active and execute:

#### Using Direct Pytest Command:
```bash
# Run all 38 unit and integration tests with verbose output (-v)
pytest tests/ -v
```

#### Using Makefile Automation:
```bash
make test
```

### 7.2 Generating Code Coverage Reports
To verify that code coverage meets or exceeds the competition requirement of **≥80%**, generate a comprehensive terminal report:

#### Using Direct Pytest Coverage Command:
```bash
# Run tests with coverage reporting across the 'app' module
pytest --cov=app --cov-report=term-missing tests/
```

#### Using Makefile Automation:
```bash
make coverage
```

**Understanding Test Execution:**  
The test suite utilizes deterministic offline mocking via `unittest.mock`. Calls to HuggingFace transformer pipelines (`facebook/bart-large-mnli`, `gpt2`) and external Wikipedia HTTP endpoints are intercepted and replaced with synthetic mock responses. This ensures that the entire 38-test suite executes cleanly in **under 3 seconds** during CI/CD pipeline runs.

---

## 8. Comprehensive Troubleshooting Guide

Below is a structured diagnostic manual addressing common engineering errors, environmental discrepancies, and deployment bottlenecks encountered during local execution or competition evaluation.

### 8.1 Port Collisions (HTTP 500 / Address Already in Use)
- **Symptom:** Uvicorn or Streamlit crashes during startup with error message: `[Errno 98] Address already in use` or `Only one usage of each socket address is normally permitted (port 8000 / 8501)`.
- **Root Cause:** Another background process, zombie container, or previously running instance is currently listening on port 8000 or 8501.
- **Diagnostic & Resolution Commands:**

#### On Windows (PowerShell / Command Prompt):
```powershell
# 1. Identify the Process ID (PID) utilizing port 8000 or 8501
netstat -ano | findstr :8000
netstat -ano | findstr :8501

# 2. Forcefully terminate the conflicting process using its PID (replace <PID> with actual number)
taskkill /PID <PID> /F
```

#### On Linux / macOS (Terminal):
```bash
# 1. Identify and display the process listening on port 8000 or 8501
lsof -i :8000
lsof -i :8501

# 2. Kill the conflicting process immediately using its PID
kill -9 <PID>
```

### 8.2 Memory Exhaustion & OOM Errors During Model Load
- **Symptom:** Application freezes during startup, or terminal outputs `MemoryError`, `Killed (SIGKILL)`, or `RuntimeError: Out of memory` when initializing `app/services/ai_service.py`.
- **Root Cause:** Loading HuggingFace `facebook/bart-large-mnli` (~1.6GB weights) and `gpt2` (~500MB weights) simultaneously into memory exceeds available physical RAM or system swap space.
- **Resolution Strategies:**
  1. **Close Memory-Intensive Applications:** Ensure at least 4.5 GB of free physical RAM is available before launching the backend server. Close background browser tabs or virtual machines.
  2. **Increase System Pagefile / Virtual Memory (Windows):** Navigate to *System Properties -> Advanced -> Performance Settings -> Advanced -> Virtual Memory*, and increase paging file size to at least 16,384 MB (16 GB).
  3. **Increase Linux / WSL2 Swap Space:** Allocate additional swap memory:
     ```bash
     sudo fallocate -l 8G /swapfile
     sudo chmod 600 /swapfile
     sudo mkswap /swapfile
     sudo swapon /swapfile
     ```
  4. **Docker Desktop Memory Allocation:** If running via Docker Desktop on macOS or Windows, open Docker Settings -> Resources -> Memory, and increase container RAM allocation from default 2GB to **8 GB minimum**.

### 8.3 Virtual Environment & Missing Modules (`ModuleNotFoundError`)
- **Symptom:** Executing `python app/main.py`, `uvicorn`, or `pytest` results in: `ModuleNotFoundError: No module named 'fastapi'` or `No module named 'streamlit'`.
- **Root Cause:** The command is being executed using the system-wide Python interpreter rather than the isolated virtual environment where dependencies were installed.
- **Resolution:**
  1. Verify that your terminal prompt displays the `(venv)` prefix.
  2. If missing, activate the virtual environment explicitly:
     - Windows: `.\venv\Scripts\activate`
     - Linux/Mac: `source venv/bin/activate`
  3. Verify which Python binary is currently active:
     - Windows: `where python` (Should point to `...\networking-assistant\venv\Scripts\python.exe`)
     - Linux/Mac: `which python` (Should point to `.../networking-assistant/venv/bin/python`)

### 8.4 HuggingFace Model Download Timeouts & SSL Certificate Errors
- **Symptom:** During initial startup, downloading model weights from HuggingFace Hub fails with `requests.exceptions.ReadTimeout`, `HTTPSConnectionPool(host='huggingface.co', port=443): Max retries exceeded`, or `SSLError`.
- **Root Cause:** Slow internet bandwidth, strict corporate firewall SSL inspection, or temporary HuggingFace Hub CDN degradation.
- **Resolution:**
  1. **Increase Download Timeout:** Set environment variable to extend socket timeout before running Uvicorn:
     ```bash
     export HF_HUB_ENABLE_HF_TRANSFER=1
     export HF_HUB_DOWNLOAD_TIMEOUT=300
     ```
  2. **Pre-Download Models via Python Interactive Shell:**
     ```python
     # Run in terminal: python -c "..."
     import transformers
     transformers.pipeline("zero-shot-classification", model="facebook/bart-large-mnli")
     transformers.pipeline("text-generation", model="gpt2")
     ```
  3. **Enable Offline Mode (Post-Download):** Once models are cached locally in `./models_cache`, set `HF_HUB_OFFLINE=1` in your `.env` file to force transformers to load exclusively from local disk without attempting network requests.

### 8.5 Wikipedia API Rate Limiting & HTTP 403 Forbidden Errors
- **Symptom:** Fact-checking queries in the UI return fallback messages, and backend logs display: `HTTPException 403 Forbidden: Wikipedia REST API rejected request` or `HTTP 429 Too Many Requests`.
- **Root Cause:** Wikipedia's REST API strictly mandates that all automated clients supply a descriptive, unique `User-Agent` HTTP header. Default requests sent with standard `requests` or `httpx` headers are blocked by Wikipedia's Cloudflare DDoS protection.
- **Resolution:**
  1. Open your `.env` file and ensure `WIKIPEDIA_USER_AGENT` is explicitly defined with a valid email or contact URL:
     ```ini
     WIKIPEDIA_USER_AGENT="PersonalizedNetworkingAssistant/1.0 (mailto:your-email@example.com)"
     ```
  2. Restart the FastAPI backend server to apply the updated configuration. The custom User-Agent header will be automatically injected into all outbound Wikipedia REST API requests in `app/services/fact_check.py`.
