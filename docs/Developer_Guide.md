# Personalized Networking Assistant: Developer Guide( Mehboob Ali Mohammed )

![Screenshot Placeholder: Developer Architecture Banner](../images/banner.png)

---

## Table of Contents
1. [Complete Project Structure & Architecture](#1-complete-project-structure--architecture)
2. [Coding Standards & Best Practices](#2-coding-standards--best-practices)
   - [2.1 PEP 8 & Formatting](#21-pep-8--formatting)
   - [2.2 Type Hinting](#22-type-hinting)
   - [2.3 Google-Style Docstrings](#23-google-style-docstrings)
   - [2.4 Pydantic v2 Validation](#24-pydantic-v2-validation)
3. [Contribution Guide & Git Workflow](#3-contribution-guide--git-workflow)
   - [3.1 Branching Strategy](#31-branching-strategy)
   - [3.2 Pull Request (PR) Process](#32-pull-request-pr-process)
   - [3.3 Code Review Standards](#33-code-review-standards)
4. [Running Locally (Development Setup)](#4-running-locally-development-setup)
   - [4.1 Backend Setup (FastAPI)](#41-backend-setup-fastapi)
   - [4.2 Frontend Setup (Streamlit)](#42-frontend-setup-streamlit)
5. [Debugging & Testing Techniques](#5-debugging--testing-techniques)
   - [5.1 Interactive API Documentation](#51-interactive-api-documentation)
   - [5.2 Logging Configuration](#52-logging-configuration)
   - [5.3 Unit & Integration Testing (pytest & Mocking)](#53-unit--integration-testing-pytest--mocking)
6. [Deployment & Container Orchestration](#6-deployment--container-orchestration)
   - [6.1 Docker Containerization](#61-docker-containerization)
   - [6.2 Docker Compose Orchestration](#62-docker-compose-orchestration)
   - [6.3 Automation via Makefile](#63-automation-via-makefile)

---

## 1. Complete Project Structure & Architecture

The **Personalized Networking Assistant** follows a clean, modular **3-Tier Architecture** separating presentation (Streamlit), business logic & AI orchestration (FastAPI), and persistent storage (Atomic JSON).

```
networking-assistant/
├── app/                        # Backend API & AI/ML Service Layer
│   ├── __init__.py             # Package initializer
│   ├── main.py                 # FastAPI application entry point, CORS, lifecycle events
│   ├── config.py               # Settings management via Pydantic-Settings (pydantic_settings)
│   ├── models/                 # Data Models Layer
│   │   ├── __init__.py
│   │   └── schemas.py          # Pydantic v2 request/response validation schemas
│   ├── routes/                 # API Routing Layer
│   │   ├── __init__.py
│   │   └── api.py              # Defines all 7 REST endpoints under /api/v1
│   └── services/               # Core Business Logic & AI/ML Engines
│       ├── __init__.py
│       ├── nlp_service.py      # Lazy-loaded BART zero-shot & GPT-2 generation engines
│       ├── fact_checker.py     # Wikipedia REST API client & confidence scoring
│       └── storage.py          # Thread-safe atomic JSON persistence engine
├── data/                       # Local Atomic Persistence Layer
│   ├── history.json            # Historical event analysis & generated starters (UUIDv4 keyed)
│   └── feedback.json           # User ratings & sentiment logs (👍/👎 and 1-5 stars)
├── docs/                       # Project Documentation & Architecture Artifacts
│   ├── images/                 # Screenshot placeholders & architecture diagrams
│   ├── User_Guide.md           # End-user operational manual & troubleshooting
│   ├── Developer_Guide.md      # Engineering setup, standards, & contribution guide
│   └── Project_Report.md       # Academic/Industry competition technical report
├── frontend/                   # Frontend Presentation Layer (Streamlit)
│   ├── __init__.py
│   ├── streamlit_app.py        # Main Streamlit UI entry point & multi-page navigation
│   ├── dashboard.py            # Visual analytics, bar charts, & sentiment metrics
│   └── history.py              # Historical interaction browser with search & pagination
├── tests/                      # Automated Quality Assurance Suite (38 tests)
│   ├── __init__.py
│   ├── test_api.py             # Integration tests for FastAPI endpoints
│   ├── test_services.py        # Unit tests for BART, GPT-2, and Wikipedia fact-checker
│   └── test_storage.py         # Concurrency & atomic file persistence verification
├── .env                        # Active local environment variables
├── .env.example                # Template for required environment configurations
├── .gitignore                  # Git exclusion rules for venv, cache, and compiled files
├── CHANGELOG.md                # Version history & release notes
├── CONTRIBUTING.md             # Quick-start contribution summary
├── Dockerfile                  # Multi-stage production build for FastAPI backend
├── Dockerfile.frontend         # Production build for Streamlit UI
├── docker-compose.yml          # Container orchestration linking API, UI, and volumes
├── LICENSE                     # MIT Open Source License
├── Makefile                    # Developer automation shortcuts (build, test, clean)
├── pytest.ini                  # Pytest configuration, coverage thresholds, & markers
├── requirements.txt            # Python dependencies (FastAPI, Streamlit, Transformers, Pytest)
└── README.md                   # Root project documentation & pitch
```

### Key Component Roles
- **`app/main.py`**: Initializes the FastAPI application, sets up CORS middleware for frontend communication, and registers API routers.
- **`app/config.py`**: Implements robust configuration management using `pydantic-settings`, loading environment variables with automatic type validation.
- **`app/models/schemas.py`**: Houses Pydantic v2 models (e.g., `EventRequest`, `StarterResponse`, `FactCheckRequest`, `FeedbackSubmission`) enforcing strict data validation.
- **`app/routes/api.py`**: Maps HTTP methods to service layer execution across 7 core endpoints: `GET /`, `POST /api/v1/analyze-event`, `POST /api/v1/generate-conversation`, `GET /api/v1/fact-check`, `GET /api/v1/history`, `POST /api/v1/feedback`, and `GET /api/v1/feedback`.
- **`app/services/`**: Encapsulates AI/ML model inference using a thread-safe **Lazy-Loaded Singleton Pattern** to prevent excessive memory consumption during startup.
- **`frontend/`**: Implements a reactive, stateful multi-tab interface using Streamlit, communicating with the backend via asynchronous HTTP requests.

---

## 2. Coding Standards & Best Practices

To maintain code quality, readability, and reliability across our competition submission, all contributors must adhere to strict engineering standards.

### 2.1 PEP 8 & Formatting
All Python code must adhere to the **PEP 8** style guide. We enforce automated formatting and linting using `flake8` and `black`:
- **Line Length:** Maximum 88 characters per line (standard Black formatting).
- **Indentation:** 4 spaces per indentation level (no tabs).
- **Naming Conventions:**
  - `snake_case` for variables, functions, and module names.
  - `PascalCase` for class names and Pydantic schemas.
  - `UPPER_SNAKE_CASE` for module-level constants and configuration flags.

### 2.2 Type Hinting
Every function signature, method, and return type across the backend and frontend **must include explicit Python type hints** (`typing` / built-in generics). This enables static analysis via `mypy` and self-documenting code.

```python
from typing import List, Optional, Dict, Any
from pydantic import BaseModel

def extract_themes_from_text(
    description: str, 
    candidate_labels: Optional[List[str]] = None
) -> Dict[str, Any]:
    # Implementation details
    pass
```

### 2.3 Google-Style Docstrings
We mandate **Google-style docstrings** for all modules, classes, and functions. Docstrings must clearly articulate purpose, arguments, return values, and potential exceptions.

```python
class FactCheckerService:
    """Service class for verifying technical claims using Wikipedia REST API.
    
    Attributes:
        base_url (str): The endpoint URL for the Wikipedia summary API.
        user_agent (str): Custom User-Agent string to adhere to API etiquette.
    """

    def verify_claim(self, query: str) -> dict[str, str]:
        """Queries Wikipedia to verify a technical claim and computes confidence.

        Args:
            query: The term, claim, or entity name to search and verify.

        Returns:
            A dictionary containing:
                - 'summary': Canonical summary extracted from Wikipedia.
                - 'confidence': Automated score ('high', 'medium', or 'low').
                - 'source_url': Direct URL to the reference Wikipedia article.

        Raises:
            ConnectionError: If network connectivity fails during API execution.
        """
        pass
```

### 2.4 Pydantic v2 Validation
All API data exchange must be validated using **Pydantic v2**. Never pass raw dictionaries or unvalidated JSON payloads between layers.

```python
from pydantic import BaseModel, Field, field_validator
from typing import List

class EventRequest(BaseModel):
    """Schema for incoming event analysis requests."""
    description: str = Field(..., min_length=10, max_length=2000, description="Event summary")
    target_role: str = Field(default="Attendee", description="User professional role")

    @field_validator("description")
    @classmethod
    def strip_whitespace(cls, value: str) -> str:
        return value.strip()
```

---

## 3. Contribution Guide & Git Workflow

Our team operates under a structured **Git Feature-Branch Workflow** to ensure code stability and seamless integration.

### 3.1 Branching Strategy
- **`main`**: Production-ready code submitted for competition evaluation. Protected branch; direct commits are strictly prohibited.
- **`develop`**: Primary integration branch for ongoing feature development.
- **`feature/<feature-name>`**: Branched from `develop` for individual features (e.g., `feature/bart-theme-extraction`, `feature/streamlit-dashboard`).
- **`bugfix/<bug-name>`**: Branched from `develop` or `main` to resolve identified defects (e.g., `bugfix/wikipedia-timeout-fallback`).
- **`release/<version>`**: Final stabilization and documentation polishing prior to major milestones.

### 3.2 Pull Request (PR) Process
1. **Create Branch:** `git checkout -b feature/your-feature-name develop`
2. **Commit Changes:** Use descriptive, atomic commit messages following the Conventional Commits specification:
   - `feat: implement zero-shot classification using BART`
   - `fix: handle Wikipedia HTTP 404 in fallback mode`
   - `test: add 5 unit tests for storage persistence`
   - `docs: update developer guide with debugging steps`
3. **Push & Open PR:** Push your branch and open a Pull Request against `develop`.
4. **PR Template Requirements:** Every PR must include:
   - A clear summary of changes implemented.
   - Reference to the issue or task ID being addressed.
   - Screenshot or terminal output proving local test verification.
   - Confirmation that all 38 pytest tests pass with $\ge 80\%$ code coverage.

### 3.3 Code Review Standards
- Every PR requires at least **one peer approval** from a team member (`teamLead` or peer `member`) before merging.
- Automated CI/CD pipelines must pass (linting, type checking, pytest suite).
- Reviewers must check for architectural alignment, adherence to the lazy-loaded singleton pattern, and proper error handling.

---

## 4. Running Locally (Development Setup)

Follow these steps to set up a complete local development environment without Docker.

### 4.1 Backend Setup (FastAPI)
1. **Open Terminal & Navigate to Project Root:**
   ```powershell
   cd C:\Users\ADMIN\.gemini\antigravity\scratch\networking-assistant
   ```
2. **Create & Activate Python Virtual Environment:**
   ```powershell
   python -m venv venv
   .\venv\Scripts\Activate.ps1
   ```
3. **Install Required Dependencies:**
   ```powershell
   pip install --upgrade pip
   pip install -r requirements.txt
   ```
4. **Configure Environment Variables:**
   Copy `.env.example` to `.env` and adjust settings if needed:
   ```powershell
   copy .env.example .env
   ```
5. **Launch FastAPI Development Server:**
   ```powershell
   uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
   ```
   *The backend is now live at `http://localhost:8000`.*

### 4.2 Frontend Setup (Streamlit)
1. **Open a Second Terminal Window** and activate the virtual environment:
   ```powershell
   cd C:\Users\ADMIN\.gemini\antigravity\scratch\networking-assistant
   .\venv\Scripts\Activate.ps1
   ```
2. **Launch Streamlit Application:**
   ```powershell
   streamlit run frontend/streamlit_app.py --server.port 8501
   ```
   *The interactive frontend will automatically open in your browser at `http://localhost:8501`.*

---

## 5. Debugging & Testing Techniques

### 5.1 Interactive API Documentation
FastAPI automatically generates live, interactive OpenAPI documentation. Use these endpoints to test backend routes independently from the Streamlit frontend:
- **Swagger UI (`/docs`):** Navigate to `http://localhost:8000/docs`. You can execute API calls directly in your browser, inspect JSON schemas, and test responses.
- **ReDoc (`/redoc`):** Navigate to `http://localhost:8000/redoc` for clean, hierarchical documentation of schemas and endpoints.

### 5.2 Logging Configuration
The backend implements structured logging using Python's built-in `logging` module. Logs are formatted to display timestamps, log levels, module names, and diagnostic messages.

```python
import logging
import sys

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    handlers=[
        logging.StreamHandler(sys.stdout),
        logging.FileHandler("app.log")
    ]
)
logger = logging.getLogger("networking_assistant")
```
- **Debug Mode:** To inspect verbose AI model loading steps and prompt tokenization, change `level=logging.INFO` to `level=logging.DEBUG` in `app/config.py` or `.env`.

### 5.3 Unit & Integration Testing (pytest & Mocking)
Our project maintains a comprehensive automated test suite consisting of **38 unit and integration tests** with $\ge 80\%$ code coverage.

#### Running the Test Suite:
```powershell
# Run all tests with detailed output
pytest -v

# Run tests with code coverage report
pytest --cov=app --cov-report=term-missing tests/
```

#### Mocking Strategy (Offline Testing):
To ensure tests execute rapidly (under 5 seconds) and reliably without downloading multi-gigabyte Hugging Face models or requiring internet connectivity, we extensively utilize `unittest.mock` and `pytest` fixtures.

```python
from unittest.mock import patch, MagicMock
import pytest
from app.services.nlp_service import NLPService

@pytest.fixture
def mock_nlp_service():
    """Fixture to mock heavy BART and GPT-2 models during testing."""
    with patch("app.services.nlp_service.pipeline") as mock_pipeline:
        # Mock BART classification output
        mock_classifier = MagicMock()
        mock_classifier.return_value = {
            "labels": ["Artificial Intelligence", "Networking"],
            "scores": [0.89, 0.75]
        }
        
        # Mock GPT-2 text generation output
        mock_generator = MagicMock()
        mock_generator.return_value = [
            {"generated_text": "Hello! I saw your work in Artificial Intelligence..."}
        ]
        
        mock_pipeline.side_effect = lambda task, **kwargs: (
            mock_classifier if task == "zero-shot-classification" else mock_generator
        )
        yield NLPService()

def test_theme_extraction_mocked(mock_nlp_service):
    themes = mock_nlp_service.extract_themes("AI and Robotics summit")
    assert "Artificial Intelligence" in themes
```

---

## 6. Deployment & Container Orchestration

For production deployments, grading evaluations, and reproducible demos, the application is fully Dockerized.

### 6.1 Docker Containerization
We utilize separate optimized Dockerfiles for the API and frontend:
- **`Dockerfile` (Backend API):** Built on `python:3.11-slim`. Pre-installs PyTorch CPU dependencies and caches Hugging Face model weights to accelerate container startup.
- **`Dockerfile.frontend` (Streamlit UI):** Lightweight container exposing port 8501, configured with health checks to ensure API connectivity.

### 6.2 Docker Compose Orchestration
The `docker-compose.yml` file orchestrates both containers, establishes internal networking, and mounts persistent storage volumes.

```yaml
version: '3.8'

services:
  backend-api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: networking_backend
    ports:
      - "8000:8000"
    volumes:
      - ./data:/app/data
    environment:
      - ENV=production
      - LOG_LEVEL=info
    restart: unless-stopped

  frontend-ui:
    build:
      context: .
      dockerfile: Dockerfile.frontend
    container_name: networking_frontend
    ports:
      - "8501:8501"
    environment:
      - BACKEND_URL=http://backend-api:8000
    depends_on:
      - backend-api
    restart: unless-stopped
```

#### Launching with Docker Compose:
```powershell
# Build and start all services in detached mode
docker-compose up -d --build

# View real-time container logs
docker-compose logs -f

# Stop and remove containers
docker-compose down
```

### 6.3 Automation via Makefile
To streamline engineering workflows, we provide a `Makefile` with standard automation shortcuts.

| Command | Action Performed |
| :--- | :--- |
| `make build` | Builds Docker images for both `backend-api` and `frontend-ui`. |
| `make run` | Starts the application locally using Docker Compose (`docker-compose up -d`). |
| `make stop` | Stops all running project containers (`docker-compose down`). |
| `make test` | Executes the full 38-test pytest suite with coverage reporting inside a container or local venv. |
| `make lint` | Runs code quality checks (`flake8`, `black --check`, `mypy`). |
| `make clean` | Removes temporary files, `.pytest_cache`, `__pycache__`, and test artifacts. |

#### Example Usage:
```powershell
# Build project and run tests automatically
make build
make test
```
