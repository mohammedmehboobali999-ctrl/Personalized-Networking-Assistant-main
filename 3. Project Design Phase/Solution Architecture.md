# Solution Architecture
## Personalized Networking Assistant

**Project:** Personalized Networking Assistant  
**Developer:** mehboob ali mohammed  
**Team Members:** mehboob ali mohammed  
**Version:** 1.0  
**Date:** July 2026  

---

## 1. Executive Summary

This document defines the comprehensive software architecture for the **Personalized Networking Assistant**. The application employs a modern **three-tier architecture** separating presentation, business logic, and data persistence. Built around a **FastAPI** REST backend and a **Streamlit** interactive frontend, the platform integrates transformer-based NLP models (**BART-large-mnli** for zero-shot classification and **GPT-2 Small** for generative dialogue) alongside real-time external knowledge verification via the **Wikipedia REST API**.

---

## 2. Architectural Layers & Overview

The system is structured into three primary architectural layers:

1. **Presentation Layer (Frontend):** A multi-page interactive web interface built with Streamlit (Python 3.11+). It manages user sessions, UI rendering, form validation, and real-time visual feedback without requiring JavaScript or custom CSS.
2. **API & Business Logic Layer (Backend):** A high-performance asynchronous REST API powered by FastAPI and Uvicorn. It handles routing, request validation via Pydantic v2, orchestration of AI models, and error handling.
3. **Service & Data Persistence Layer:** A modular subsystem responsible for AI model inference (HuggingFace Transformers), network calls to external knowledge bases (Wikipedia API), and atomic read-modify-write file operations against local JSON data stores (`history.json` and `feedback.json`).

```
+-----------------------------------------------------------------------------------+
|                              PRESENTATION LAYER                                   |
|                                                                                   |
|   +---------------------------------------------------------------------------+   |
|   |                       Streamlit Web Application                           |   |
|   |   +-----------------------+  +--------------------+  +----------------+   |   |
|   |   |  Main App (Generate)  |  | Analytics Dashboard|  | History Review |   |   |
|   |   +-----------------------+  +--------------------+  +----------------+   |   |
|   +---------------------------------------------------------------------------+   |
+-----------------------------------------+-----------------------------------------+
                                          | HTTP REST / JSON (Port 8501 -> 8000)
+-----------------------------------------v-----------------------------------------+
|                              API & ROUTING LAYER                                  |
|                                                                                   |
|   +---------------------------------------------------------------------------+   |
|   |                      FastAPI ASGI Server (Uvicorn)                        |   |
|   |   +-------------------------------------------------------------------+   |   |
|   |   |                   Pydantic v2 Request Validation                  |   |   |
|   |   +-------------------------------------------------------------------+   |   |
|   |   |  Routes: /analyze-event | /generate-conversation | /fact-check    |   |   |
|   |   |          /history       | /feedback              | /health        |   |   |
|   |   +-------------------------------------------------------------------+   |   |
|   +---------------------------------------------------------------------------+   |
+-----------------------------------------+-----------------------------------------+
                                          | Python Function Calls / Models
+-----------------------------------------v-----------------------------------------+
|                        SERVICE & DATA PERSISTENCE LAYER                           |
|                                                                                   |
|   +-----------------------+  +------------------------+  +--------------------+   |
|   |      NLP Service      |  |   Generation Service   |  | Fact-Check Service |   |
|   | (BART-large-mnli HF)  |  |   (GPT-2 Small HF)     |  | (Wikipedia REST)   |   |
|   +-----------------------+  +------------------------+  +--------------------+   |
|                                                                                   |
|   +-----------------------+  +------------------------+  +--------------------+   |
|   |    History Logger     |  |    Feedback Logger     |  | Storage Service    |   |
|   +-----------------------+  +------------------------+  +--------------------+   |
|                                          |                                        |
|                                          v                                        |
|                              +------------------------+                           |
|                              |  Local File System     |                           |
|                              |  - data/history.json   |                           |
|                              |  - data/feedback.json  |                           |
|                              +------------------------+                           |
+-----------------------------------------------------------------------------------+
```

---

## 3. Directory Structure & Component Inventory

```
networking-assistant/
├── app/
│   ├── __init__.py
│   ├── main.py                  # FastAPI application setup, CORS, route inclusion, lifespan
│   ├── config.py                # Pydantic BaseSettings loading from .env
│   ├── models/
│   │   ├── __init__.py
│   │   ├── schemas.py           # Pydantic request/response data models
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── api.py               # API route definitions and endpoint handlers
│   ├── services/
│   │   ├── __init__.py
│   │   ├── nlp_service.py       # BART zero-shot theme classification engine
│   │   ├── generation_service.py# GPT-2 text generation engine & post-processing
│   │   ├── storage_service.py   # Atomic file I/O for JSON persistence
│   │   ├── event_analyzer.py    # SmartBridge alias interface for event analysis
│   │   ├── topic_generator.py   # SmartBridge alias interface for topic generation
│   │   ├── fact_checker.py      # Wikipedia REST API client wrapper
│   │   ├── history_logger.py    # SmartBridge alias interface for conversation history
│   │   └── feedback_logger.py   # SmartBridge alias interface for feedback logging
├── frontend/
│   ├── __init__.py
│   ├── streamlit_app.py         # Main entry point for Streamlit frontend
│   ├── pages/
│   │   ├── dashboard.py         # Analytics & statistics dashboard
│   │   └── history.py           # Interactive history & feedback browser
├── data/
│   ├── history.json             # Persistent JSON store for generated sessions
│   └── feedback.json            # Persistent JSON store for user ratings & reviews
├── tests/
│   ├── test_event_analyzer.py   # Unit tests for BART theme extraction
│   ├── test_topic_generator.py  # Unit tests for GPT-2 generation & cleanup
│   ├── test_fact_checker.py     # Unit tests for Wikipedia API (mocked network)
│   ├── test_api.py              # Integration tests for FastAPI endpoints
│   └── ...                      # Full 38-test suite
├── Dockerfile                   # Backend Docker container specification
├── Dockerfile.frontend          # Frontend Docker container specification
├── docker-compose.yml           # Multi-container orchestration
├── Makefile                     # Developer command automation
├── requirements.txt             # Locked Python dependency specifications
└── README.md                    # Project documentation & setup instructions
```

---

## 4. API Design Reference

The backend exposes seven RESTful endpoints under the `/api/v1` prefix (and root `/`). All request and response bodies are validated using Pydantic schemas.

| Method | Endpoint | Description | Request Body / Params | Response Status & Body |
|---|---|---|---|---|
| `GET` | `/` | System health check | None | `200 OK` — `{"status": "ok", "message": "API is running"}` |
| `POST` | `/api/v1/analyze-event` | Extract top-3 themes from event description | `{"event_description": "string (>=10 chars)"}` | `200 OK` — `{"themes": ["AI", "sustainability", "tech"]}` |
| `POST` | `/api/v1/generate-conversation` | Generate personalized conversation starters | `{"themes": ["..."], "user_bio": "...", "num_starters": 3}` | `200 OK` — `{"session_id": "uuid", "starters": ["..."], "themes_used": ["..."]}` |
| `GET` | `/api/v1/fact-check` | Verify claim/topic via Wikipedia API | Query param: `?query=string` | `200 OK` — `{"found": true, "extract": "...", "url": "...", "confidence": "high"}` |
| `GET` | `/api/v1/history` | Retrieve paginated conversation history | Query params: `?page=1&page_size=5&search=kw` | `200 OK` — `{"items": [...], "total": 10, "page": 1, "pages": 2}` |
| `POST` | `/api/v1/feedback` | Submit user rating for a starter | `{"session_id": "uuid", "rating": 4, "comment": "good", "suggestion": "...", "action": "like"}` | `200 OK` — `{"status": "success", "message": "Feedback recorded"}` |
| `GET` | `/api/v1/feedback` | Retrieve recent feedback entries | Query param: `?limit=10` | `200 OK` — `[{"suggestion": "...", "action": "like", "timestamp": "..."}]` |

---

## 5. Service Layer & Design Principles

The service layer is engineered following strict software engineering standards:

1. **Single Responsibility Principle (SRP):** Each service module encapsulates exactly one domain. `nlp_service.py` only handles classification; `generation_service.py` only handles text generation; `storage_service.py` only handles disk I/O.
2. **Lazy Singleton Pattern for ML Models:** To prevent massive memory spikes and slow startup times, machine learning pipelines (`transformers.pipeline`) are initialized via lazy loading. The model weights are loaded into CPU memory only upon the first request and are subsequently cached in thread-safe global singletons.
3. **Defensive External Communication:** All calls to external APIs (`requests.get` to Wikipedia) are wrapped in comprehensive try-except blocks with explicit timeouts (`timeout=10.0`). If an external service is unavailable or times out, the system gracefully falls back to a structured default dictionary without throwing unhandled HTTP 500 errors.
4. **Atomic Read-Modify-Write Persistence:** To prevent race conditions and JSON file corruption during concurrent read/write operations, all file modifications read the full JSON array into memory, append the timestamped record, and write out to disk atomically using `pathlib.Path`.

---

## 6. Data Architecture & Schemas

The application utilizes two primary JSON schemas persisted in the `/data` directory:

### `data/history.json` Schema
```json
[
  {
    "session_id": "8f9d3a12-4c6b-4e11-9a23-8b7c6d5e4f3a",
    "timestamp": "2026-07-06T14:30:00.000000",
    "event_description": "AI for Sustainable Cities Summit 2026",
    "themes": ["AI", "sustainability", "technology"],
    "user_bio": "Software engineer interested in smart grids",
    "starters": [
      "What brought you to explore AI applications for urban infrastructure?",
      "I'd love to hear your thoughts on balancing IoT data growth with grid sustainability.",
      "Are there specific machine learning tools you use for urban planning?"
    ]
  }
]
```

### `data/feedback.json` Schema
```json
[
  {
    "id": "c3e1a90b-2d4e-4f6a-8b1c-9a0b1c2d3e4f",
    "session_id": "8f9d3a12-4c6b-4e11-9a23-8b7c6d5e4f3a",
    "suggestion": "What brought you to explore AI applications for urban infrastructure?",
    "action": "like",
    "rating": 5,
    "comment": "Very natural and relevant question!",
    "timestamp": "2026-07-06T14:35:12.123456"
  }
]
```

---

## 7. Security & Deployment Architecture

### Security Controls
- **Environment Isolation:** Sensitive configurations and environment variables are loaded via `pydantic-settings` from `.env`. The `.env` file is explicitly excluded from version control via `.gitignore`, committing only `.env.example`.
- **Input Sanitization & Validation:** Pydantic v2 enforces strict type checking and length constraints on all incoming HTTP payloads, preventing injection attacks and malformed data crashes.
- **No External Data Leakage:** User bios and event descriptions are processed entirely locally on the server's CPU via open-source HuggingFace models. No user prompts or personal data are transmitted to third-party LLM providers (e.g., OpenAI or Anthropic).

### Deployment & Containerization
The system is containerized using Docker and Docker Compose for reproducible, cross-platform deployments across Windows, macOS, and Linux:
- **Backend Container (`api`):** Built from `python:3.11-slim`, exposes port `8000`, runs Uvicorn ASGI server.
- **Frontend Container (`frontend`):** Built from `python:3.11-slim`, exposes port `8501`, runs Streamlit web server.
- **Volume Mounting:** The `./data` directory is mounted as a persistent Docker volume to ensure user history and feedback survive container restarts.
- **Internal Networking:** The Streamlit container communicates with the FastAPI container via Docker's internal DNS resolution (`http://api:8000`).

---

## 8. Scalability & Future Architecture Roadmap

While the MVP architecture is optimized for lightweight local and single-server execution, it is designed for seamless horizontal scaling:

| Layer | Current MVP Architecture | Future Enterprise Scalability Plan |
|---|---|---|
| **Data Storage** | Local JSON files (`history.json`, `feedback.json`) | Migration to PostgreSQL or AWS Aurora with SQLalchemy ORM |
| **Model Inference** | Local CPU inference via HuggingFace Transformers | Dedicated GPU inference servers (Nvidia Triton / Triton Inference Server) or AWS SageMaker endpoints |
| **Caching Layer** | In-memory Python singleton dicts | Distributed Redis caching for frequent Wikipedia queries and common event themes |
| **Asynchronous Tasks** | Synchronous HTTP request-response cycle | Celery / RabbitMQ background task queues for heavy NLP batch processing |
| **Load Balancing** | Single Uvicorn worker | Nginx reverse proxy / AWS ALB routing across multiple Uvicorn worker pods in Kubernetes (EKS) |
