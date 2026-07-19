# Project Planning Document
## Personalized Networking Assistant

---

| Field             | Details                                            |
|-------------------|----------------------------------------------------|
| **Project Name**  | Personalized Networking Assistant                  |
| **Version**       | 1.0                                                |
| **Date**          | July 7, 2026                                       |
| **Developer**     | mehboob ali mohammed                               |
| **Team Members**  | mehboob ali mohammed |
| **Tech Stack**    | FastAPI · Streamlit · BART · GPT-2 · Wikipedia API |
| **Document Type** | Project Planning                                   |

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Sprint Plan — Timeline](#2-sprint-plan--timeline)
3. [Work Breakdown Structure (WBS)](#3-work-breakdown-structure-wbs)
4. [Team Responsibilities](#4-team-responsibilities)
5. [Resource Allocation](#5-resource-allocation)
6. [Risk Management](#6-risk-management)
7. [Milestones and Deliverables](#7-milestones-and-deliverables)
8. [Definition of Done](#8-definition-of-done)
9. [Planning Tools](#9-planning-tools)
10. [Approval](#10-approval)

---

## 1. Project Overview

The **Personalized Networking Assistant** is an AI-powered web application designed to help professionals build meaningful connections by generating personalized outreach messages, summarizing contact backgrounds, and validating factual claims. The system combines state-of-the-art NLP models (Facebook BART for summarization, GPT-2 for message generation) with real-time information retrieval via the Wikipedia API, all served through a high-performance FastAPI backend and an interactive Streamlit frontend.

### Objectives

- Deliver an end-to-end AI assistant that automates professional networking outreach.
- Provide summarized, Wikipedia-enriched contact profiles to personalize messages.
- Ensure factual accuracy of generated content through an integrated fact-checking service.
- Ship a containerized, production-ready application within an 8-week development cycle (4 sprints × 2 weeks).

### Scope

| In Scope | Out of Scope |
|----------|--------------|
| BART-based profile summarization | LinkedIn / third-party OAuth integration |
| GPT-2 personalized message generation | Real-time social media scraping |
| Wikipedia API fact-checking | Paid cloud infrastructure provisioning |
| FastAPI REST backend with SQLite/JSON storage | Mobile native application |
| Streamlit interactive frontend | Multi-language UI localization |
| Docker containerization & deployment | Enterprise SSO authentication |
| Automated test suite (pytest ≥ 80 % coverage) | |

---

## 2. Sprint Plan — Timeline

The project follows a **Scrum-based Agile** methodology with four two-week sprints totaling **8 weeks** of active development. Each sprint begins with a planning session and closes with a review and retrospective.

### High-Level Gantt Overview

```
Week:  | 1  2 | 3  4 | 5  6 | 7  8 |
       |------|------|------|------|
S1     |██████|      |      |      |  Environment, Schema, NLP Service
S2     |      |██████|      |      |  Generation, Fact-Check, Storage
S3     |      |      |██████|      |  FastAPI Routes, Streamlit, Integration
S4     |      |      |      |██████|  Testing, Docs, Deployment, Demo
```

---

### Sprint 1 — Foundation & NLP Service
**Duration:** Week 1 – Week 2
**Goal:** Establish a working development environment, define data contracts, and deliver an operational NLP summarization service.

| # | Task | Owner | Story Points | Status |
|---|------|-------|:---:|--------|
| 1.1 | Create GitHub repository, branch strategy (`main`, `develop`, feature branches) | mehboob ali mohammed | 2 | Planned |
| 1.2 | Define project folder structure and module layout | mehboob ali mohammed | 1 | Planned |
| 1.3 | Configure Python virtual environment, `requirements.txt`, `.env` template | mehboob ali mohammed | 2 | Planned |
| 1.4 | Design contact data schema (JSON + SQLite DDL) | mehboob ali mohammed | 3 | Planned |
| 1.5 | Implement BART summarization service (`services/nlp_service.py`) | mehboob ali mohammed | 8 | Planned |
| 1.6 | Write unit tests for BART service (pytest) | mehboob ali mohammed | 3 | Planned |
| 1.7 | Set up CI pipeline (GitHub Actions — lint + unit tests) | mehboob ali mohammed | 3 | Planned |
| 1.8 | Write Sprint 1 planning notes and update project board | mehboob ali mohammed | 1 | Planned |

**Sprint 1 Total Story Points:** 23
**Sprint 1 Goal Acceptance Criteria:**
- BART summarizer returns coherent summaries for sample contact bios.
- Data schema is documented and validated against sample inputs.
- GitHub Actions CI passes on every push to `develop`.

---

### Sprint 2 — Generation, Fact-Check & Storage Services
**Duration:** Week 3 – Week 4
**Goal:** Build the GPT-2 message generation service, Wikipedia-powered fact-checking service, and persistent storage layer.

| # | Task | Owner | Story Points | Status |
|---|------|-------|:---:|--------|
| 2.1 | Implement GPT-2 message generation service (`services/generation_service.py`) | mehboob ali mohammed | 8 | Planned |
| 2.2 | Build prompt engineering module for GPT-2 personalization | mehboob ali mohammed | 5 | Planned |
| 2.3 | Implement Wikipedia API wrapper (`services/wiki_service.py`) | mehboob ali mohammed | 5 | Planned |
| 2.4 | Implement fact-checking service (`services/fact_check_service.py`) | mehboob ali mohammed | 5 | Planned |
| 2.5 | Implement SQLite storage service (`services/storage_service.py`) | mehboob ali mohammed | 5 | Planned |
| 2.6 | Implement JSON file-based storage fallback | mehboob ali mohammed | 3 | Planned |
| 2.7 | Write unit & integration tests for all Sprint 2 services | mehboob ali mohammed | 5 | Planned |
| 2.8 | Model optimization — quantization / caching for BART & GPT-2 | mehboob ali mohammed | 5 | Planned |
| 2.9 | Sprint 2 review, retrospective, and updated documentation | mehboob ali mohammed | 1 | Planned |

**Sprint 2 Total Story Points:** 42
**Sprint 2 Goal Acceptance Criteria:**
- GPT-2 generates grammatically correct, context-aware networking messages.
- Fact-checker accurately flags unverified claims against Wikipedia.
- SQLite storage correctly persists and retrieves contact records.
- All services pass their test suites with ≥ 75 % coverage.

---

### Sprint 3 — FastAPI Routes, Streamlit Frontend & Integration
**Duration:** Week 5 – Week 6
**Goal:** Expose all services through a production-quality REST API and build the interactive Streamlit frontend. Perform end-to-end integration.

| # | Task | Owner | Story Points | Status |
|---|------|-------|:---:|--------|
| 3.1 | Design and implement FastAPI application structure (`main.py`, routers) | mehboob ali mohammed | 3 | Planned |
| 3.2 | Implement `/summarize` route (POST) — calls NLP service | mehboob ali mohammed | 3 | Planned |
| 3.3 | Implement `/generate-message` route (POST) — calls generation service | mehboob ali mohammed | 3 | Planned |
| 3.4 | Implement `/fact-check` route (POST) — calls fact-check service | mehboob ali mohammed | 3 | Planned |
| 3.5 | Implement `/contacts` CRUD routes (GET, POST, PUT, DELETE) | mehboob ali mohammed | 5 | Planned |
| 3.6 | Add Pydantic request/response models and input validation | mehboob ali mohammed | 3 | Planned |
| 3.7 | Build Streamlit UI — Home page and navigation | mehboob ali mohammed | 3 | Planned |
| 3.8 | Build Streamlit UI — Contact Profile & Summary page | mehboob ali mohammed | 5 | Planned |
| 3.9 | Build Streamlit UI — Message Generator page | mehboob ali mohammed | 5 | Planned |
| 3.10 | Build Streamlit UI — Fact Checker page | mehboob ali mohammed | 3 | Planned |
| 3.11 | Build Streamlit UI — Contact Manager (CRUD) page | mehboob ali mohammed | 5 | Planned |
| 3.12 | End-to-end integration testing (Streamlit ↔ FastAPI ↔ Services) | mehboob ali mohammed | 5 | Planned |
| 3.13 | Fix integration bugs identified during Sprint 3 testing | mehboob ali mohammed | 5 | Planned |

**Sprint 3 Total Story Points:** 51
**Sprint 3 Goal Acceptance Criteria:**
- All FastAPI endpoints return correct responses with proper HTTP status codes.
- Streamlit frontend successfully communicates with all FastAPI routes.
- Full user journey (enter contact → summarize → generate message → fact-check → save) works end-to-end.
- API documentation auto-generated at `/docs` (Swagger UI).

---

### Sprint 4 — Testing, Documentation, Deployment & Demo
**Duration:** Week 7 – Week 8
**Goal:** Achieve ≥ 80 % test coverage, finalize all project documentation, containerize and deploy the application, and deliver the final demo.

| # | Task | Owner | Story Points | Status |
|---|------|-------|:---:|--------|
| 4.1 | Write comprehensive pytest test suite (unit + integration + edge cases) | mehboob ali mohammed | 8 | Planned |
| 4.2 | Achieve ≥ 80 % code coverage (pytest-cov report) | mehboob ali mohammed | 5 | Planned |
| 4.3 | Performance testing — measure and document API response times | mehboob ali mohammed | 3 | Planned |
| 4.4 | Write `Dockerfile` for FastAPI service | mehboob ali mohammed | 3 | Planned |
| 4.5 | Write `Dockerfile` for Streamlit service | mehboob ali mohammed | 3 | Planned |
| 4.6 | Write `docker-compose.yml` for multi-service orchestration | mehboob ali mohammed | 3 | Planned |
| 4.7 | Deploy to local/staging environment and smoke test | mehboob ali mohammed | 3 | Planned |
| 4.8 | Write technical README with setup, usage, and API reference | mehboob ali mohammed | 3 | Planned |
| 4.9 | Write System Design Document | mehboob ali mohammed | 5 | Planned |
| 4.10 | Write API Documentation (extended Swagger descriptions + Postman collection) | mehboob ali mohammed | 3 | Planned |
| 4.11 | Finalize Project Report for SmartBridge submission | mehboob ali mohammed | 5 | Planned |
| 4.12 | Prepare and rehearse final demo presentation | mehboob ali mohammed | 3 | Planned |
| 4.13 | Deliver final demo | mehboob ali mohammed | 1 | Planned |

**Sprint 4 Total Story Points:** 48
**Sprint 4 Goal Acceptance Criteria:**
- Test coverage report shows ≥ 80 % across all modules.
- Docker Compose brings up all services successfully with a single command.
- All project documents submitted to SmartBridge portal.
- Demo successfully showcases the complete user journey.

---

### Sprint Summary Table

| Sprint | Duration | Focus Area | Story Points | Key Deliverable |
|--------|----------|------------|:---:|-----------------|
| Sprint 1 | Weeks 1–2 | Environment, Schema, NLP Service | 23 | Working BART Summarizer |
| Sprint 2 | Weeks 3–4 | Generation, Fact-Check, Storage | 42 | All Core Services |
| Sprint 3 | Weeks 5–6 | FastAPI, Streamlit, Integration | 51 | Full App (Integrated) |
| Sprint 4 | Weeks 7–8 | Testing, Docs, Deployment, Demo | 48 | Production-Ready Release |
| **Total** | **8 Weeks** | | **164** | **Deployed Application** |

---

## 3. Work Breakdown Structure (WBS)

```
Personalized Networking Assistant
│
├── 1. Project Management
│   ├── 1.1 Project Initiation
│   │   ├── 1.1.1 Define scope and objectives
│   │   ├── 1.1.2 Identify stakeholders
│   │   └── 1.1.3 Prepare project charter
│   ├── 1.2 Planning
│   │   ├── 1.2.1 Create sprint plan
│   │   ├── 1.2.2 Assign roles and responsibilities
│   │   └── 1.2.3 Set up project board (GitHub Projects)
│   └── 1.3 Monitoring & Control
│       ├── 1.3.1 Weekly stand-ups
│       ├── 1.3.2 Sprint reviews and retrospectives
│       └── 1.3.3 Risk tracking
│
├── 2. Environment & Infrastructure
│   ├── 2.1 Repository Setup
│   │   ├── 2.1.1 Initialize GitHub repository
│   │   ├── 2.1.2 Define branching strategy
│   │   └── 2.1.3 Configure .gitignore, CODEOWNERS
│   ├── 2.2 Development Environment
│   │   ├── 2.2.1 Python virtual environment
│   │   ├── 2.2.2 requirements.txt / pyproject.toml
│   │   └── 2.2.3 Environment variable management (.env)
│   └── 2.3 CI/CD Pipeline
│       ├── 2.3.1 GitHub Actions — lint (flake8/ruff)
│       ├── 2.3.2 GitHub Actions — unit test runner
│       └── 2.3.3 GitHub Actions — coverage gate
│
├── 3. Data Layer
│   ├── 3.1 Data Schema Design
│   │   ├── 3.1.1 Contact data model (JSON schema)
│   │   ├── 3.1.2 SQLite database DDL
│   │   └── 3.1.3 Pydantic models for API validation
│   └── 3.2 Storage Services
│       ├── 3.2.1 SQLite CRUD service
│       └── 3.2.2 JSON file-based storage fallback
│
├── 4. NLP & AI Services
│   ├── 4.1 Summarization Service (BART)
│   │   ├── 4.1.1 Load facebook/bart-large-cnn model
│   │   ├── 4.1.2 Implement summarize() function
│   │   └── 4.1.3 Model caching and optimization
│   ├── 4.2 Message Generation Service (GPT-2)
│   │   ├── 4.2.1 Load GPT-2 model and tokenizer
│   │   ├── 4.2.2 Prompt engineering for personalization
│   │   ├── 4.2.3 Implement generate_message() function
│   │   └── 4.2.4 Decoding strategy tuning (top-k, top-p)
│   ├── 4.3 Wikipedia Fact-Check Service
│   │   ├── 4.3.1 Wikipedia API integration
│   │   ├── 4.3.2 Claim extraction logic
│   │   └── 4.3.3 Verification and confidence scoring
│   └── 4.4 Model Optimization
│       ├── 4.4.1 Quantization (INT8)
│       ├── 4.4.2 Model warm-up on startup
│       └── 4.4.3 Response caching (in-memory LRU)
│
├── 5. Backend — FastAPI
│   ├── 5.1 Application Structure
│   │   ├── 5.1.1 main.py with lifespan events
│   │   ├── 5.1.2 Router modules per domain
│   │   └── 5.1.3 Dependency injection setup
│   ├── 5.2 API Endpoints
│   │   ├── 5.2.1 POST /summarize
│   │   ├── 5.2.2 POST /generate-message
│   │   ├── 5.2.3 POST /fact-check
│   │   ├── 5.2.4 GET  /contacts
│   │   ├── 5.2.5 POST /contacts
│   │   ├── 5.2.6 PUT  /contacts/{id}
│   │   └── 5.2.7 DELETE /contacts/{id}
│   └── 5.3 Cross-Cutting Concerns
│       ├── 5.3.1 CORS middleware
│       ├── 5.3.2 Global error handling
│       ├── 5.3.3 Request logging
│       └── 5.3.4 Auto-generated Swagger docs (/docs)
│
├── 6. Frontend — Streamlit
│   ├── 6.1 Application Shell
│   │   ├── 6.1.1 Sidebar navigation
│   │   └── 6.1.2 Session state management
│   ├── 6.2 Pages
│   │   ├── 6.2.1 Home / Dashboard page
│   │   ├── 6.2.2 Contact Profile & Summary page
│   │   ├── 6.2.3 Message Generator page
│   │   ├── 6.2.4 Fact Checker page
│   │   └── 6.2.5 Contact Manager (CRUD) page
│   └── 6.3 UX Polish
│       ├── 6.3.1 Loading spinners for AI calls
│       ├── 6.3.2 Error / warning display
│       └── 6.3.3 Copy-to-clipboard for generated messages
│
├── 7. Testing & Quality Assurance
│   ├── 7.1 Unit Tests
│   │   ├── 7.1.1 NLP service tests
│   │   ├── 7.1.2 Generation service tests
│   │   ├── 7.1.3 Fact-check service tests
│   │   └── 7.1.4 Storage service tests
│   ├── 7.2 Integration Tests
│   │   ├── 7.2.1 FastAPI endpoint tests (TestClient)
│   │   └── 7.2.2 End-to-end workflow tests
│   ├── 7.3 Performance Tests
│   │   ├── 7.3.1 API response time benchmarks
│   │   └── 7.3.2 Model inference latency measurement
│   └── 7.4 Coverage
│       └── 7.4.1 pytest-cov report (≥ 80 % target)
│
├── 8. Deployment
│   ├── 8.1 Containerization
│   │   ├── 8.1.1 Dockerfile — FastAPI service
│   │   ├── 8.1.2 Dockerfile — Streamlit service
│   │   └── 8.1.3 docker-compose.yml
│   └── 8.2 Deployment Verification
│       ├── 8.2.1 Container smoke tests
│       └── 8.2.2 Service health check endpoints
│
└── 9. Documentation
    ├── 9.1 Technical README
    ├── 9.2 System Design Document
    ├── 9.3 API Reference Documentation
    ├── 9.4 Project Planning Document (this file)
    ├── 9.5 Test Report
    └── 9.6 Final Project Report (SmartBridge)
```

---

## 4. Team Responsibilities

### 4.1 Mehboob Ali Mohammed — Team Lead
**Role:** Project Lead · System Architect · FastAPI Backend Developer

| Responsibility | Details |
|----------------|---------|
| **System Architecture** | Design the overall application architecture (microservice-style layers: services, API, frontend, storage). Maintain architecture decision records (ADRs). |
| **FastAPI Backend** | Own `main.py`, all routers, Pydantic models, CORS, error handling, and dependency injection. Implement all seven API endpoints. |
| **Integration Lead** | Ensure seamless integration between AI services and the FastAPI layer. Define internal service contracts. |
| **Wikipedia Service** | Implement `wiki_service.py` and `fact_check_service.py`. |
| **Data Schema** | Design the contact data model (JSON schema + SQLite DDL + Pydantic models). |
| **Sprint Management** | Facilitate sprint planning, daily stand-ups, reviews, and retrospectives. Maintain GitHub project board. |
| **Code Reviews** | Review and merge all pull requests to `main` and `develop`. |
| **API Documentation** | Author Swagger endpoint descriptions and maintain Postman collection. |

**Primary Files:** `main.py` · `routers/` · `models/` · `services/wiki_service.py` · `services/fact_check_service.py` · `schemas/`

---

### 4.2 Mehboob Ali Mohammed — NLP Engineer
**Role:** AI/ML Developer · NLP Services Specialist

| Responsibility | Details |
|----------------|---------|
| **BART Summarization** | Implement and tune `services/nlp_service.py` using `facebook/bart-large-cnn`. Ensure summaries are concise and contextually accurate. |
| **GPT-2 Message Generation** | Implement `services/generation_service.py` with custom prompt engineering to produce personalized, professional networking messages. |
| **Prompt Engineering** | Design and iterate on prompt templates to maximize GPT-2 output quality and relevance. |
| **Model Optimization** | Apply model quantization (INT8), implement LRU response caching, and warm-up models at service startup to minimize inference latency. |
| **Decoding Strategy Tuning** | Configure top-k sampling, top-p nucleus sampling, temperature, and repetition penalty for GPT-2. |
| **Environment Setup** | Configure Python virtual environment, `requirements.txt`, and `.env` template for reproducible setups across all team members' machines. |

**Primary Files:** `services/nlp_service.py` · `services/generation_service.py` · `utils/prompt_templates.py` · `requirements.txt`
## 5. Resource Allocation

### Human Resources

| Team Member | Role | Sprint 1 | Sprint 2 | Sprint 3 | Sprint 4 | Total Points |
|-------------|------|:---:|:---:|:---:|:---:|:---:|
| mehboob ali mohammed | Full-Stack Developer | 23 | 42 | 51 | 48 | 164 |
| **Total** | | **23** | **42** | **51** | **48** | **164** |

> **Note:** Story point allocations represent relative effort. Each sprint assumes approximately 40 available developer-hours per team member.

### Technology Resources

| Resource | Tool / Version | Purpose |
|----------|---------------|---------|
| Language | Python 3.10+ | All backend and frontend development |
| AI Framework | HuggingFace Transformers 4.x | BART + GPT-2 model loading and inference |
| Backend Framework | FastAPI 0.111+ | REST API server |
| Frontend Framework | Streamlit 1.35+ | Interactive web UI |
| External API | Wikipedia API (wikipedia-api 0.6+) | Fact-checking and knowledge retrieval |
| Database | SQLite 3 | Persistent contact storage |
| Testing | pytest 7+, pytest-cov, pytest-mock | Automated testing and coverage |
| Containerization | Docker 24+, Docker Compose v2 | Deployment packaging |
| Version Control | Git + GitHub | Source code management |
| CI/CD | GitHub Actions | Automated lint, test, coverage gate |
| Project Tracking | GitHub Projects (Kanban board) | Sprint management |
| Documentation Platform | SmartBridge Portal | Final submission |

### Infrastructure Resources

| Environment | Specification | Purpose |
|-------------|--------------|---------|
| Development | Local machine (8 GB+ RAM recommended) | Day-to-day development |
| CI | GitHub Actions (ubuntu-latest runner) | Automated testing |
| Staging / Demo | Docker Compose on local machine | Integration and demo |
| Model Storage | Local disk (BART ~1.6 GB, GPT-2 ~548 MB) | Model weights cache |

---

## 6. Risk Management

### Risk Register

| # | Risk | Category | Probability | Impact | Severity | Mitigation Strategy | Owner |
|---|------|----------|:-----------:|:------:|:--------:|---------------------|-------|
| R1 | **Model Inference Latency** — BART and GPT-2 may have response times exceeding acceptable UX thresholds (> 10 s) on CPU-only hardware. | Technical | High | High | 🔴 Critical | (1) Implement in-memory LRU caching for repeated inputs. (2) Apply INT8 quantization to reduce model size and speed up inference. (3) Warm up models at application startup. (4) Set realistic user expectations via Streamlit loading spinners and progress bars. | mehboob ali mohammed |
| R2 | **Wikipedia API Availability** — The external Wikipedia API may return rate-limit errors or be temporarily unavailable, breaking the fact-check feature. | External | Medium | Medium | 🟠 High | (1) Implement retry logic with exponential back-off (max 3 retries). (2) Add a fallback graceful response ("Fact-check service temporarily unavailable"). (3) Cache successful Wikipedia responses locally to reduce repeated calls. (4) Wrap all API calls in try/except with meaningful HTTP error responses. | mehboob ali mohammed |
| R3 | **GPT-2 Output Quality** — GPT-2 may generate off-topic, repetitive, or unprofessional messages, reducing product usefulness. | AI/Quality | High | Medium | 🟠 High | (1) Invest in iterative prompt engineering with structured templates (role, context, goal). (2) Tune decoding parameters (top-p = 0.92, top-k = 50, repetition_penalty = 1.3). (3) Implement a post-processing filter for basic quality checks. (4) Allow users to regenerate messages with a single click in the UI. | mehboob ali mohammed |
| R4 | **Integration Complexity** — Connecting Streamlit ↔ FastAPI ↔ multiple AI services may introduce subtle bugs that are difficult to debug in Sprint 3. | Technical | Medium | High | 🟠 High | (1) Define and freeze internal API contracts (Pydantic schemas) before Sprint 3 begins. (2) Write integration tests throughout Sprint 2 as services are built. (3) Use FastAPI's `TestClient` for backend integration tests before the Streamlit layer is connected. (4) Dedicate buffer time (task 3.13, 5 points) for integration bug fixes. | mehboob ali mohammed |
| R5 | **Timeline Slippage** — Team members may face unforeseen academic or personal commitments, causing sprint tasks to fall behind schedule. | Schedule | Medium | Medium | 🟡 Medium | (1) Maintain a GitHub Projects Kanban board with clear task ownership and due dates. (2) Conduct brief daily async stand-ups via messaging to flag blockers early. (3) Designate mehboob ali mohammed as decision-maker to re-prioritize tasks if scope must be cut. (4) Identify a minimum viable scope (MVP): summarize + generate + basic UI) that can be delivered even if edge features slip. | mehboob ali mohammed |

### Risk Severity Matrix

```
Impact ↑
High   |        | R2, R4 | R1     |
Medium |        | R5     | R3     |
Low    |        |        |        |
        +--------+--------+--------→ Probability
         Low      Medium   High
```

---

## 7. Milestones and Deliverables

| Milestone | Due Date | Deliverable(s) | Acceptance Criteria | Owner |
|-----------|----------|----------------|---------------------|-------|
| **M1 — Foundation Complete** | End of Week 2 | Working BART summarization service; GitHub repo with CI; data schema | BART service passes unit tests; CI green | Naga / Sumiya |
| **M2 — Core Services Complete** | End of Week 4 | GPT-2 service; Wikipedia fact-check service; SQLite storage; test suite for all services | All services pass tests; ≥ 75 % coverage | Naga / Sumiya / Satvika |
| **M3 — Integrated Application** | End of Week 6 | Functional FastAPI backend; Streamlit frontend; end-to-end user journey working | All API routes return correct responses; E2E demo scenario passes | All |
| **M4 — Production Release** | End of Week 8 | Docker Compose deployment; ≥ 80 % test coverage; all documentation submitted; final demo delivered | Docker Compose up succeeds; coverage ≥ 80 %; SmartBridge submission confirmed | All |

### Deliverables Checklist

| # | Deliverable | Format | Responsible | Sprint |
|---|-------------|--------|-------------|--------|
| D1 | GitHub Repository (all source code) | Git Repo | Shaik Sumiya Zainab | All |
| D2 | `services/nlp_service.py` (BART) | Python Module | Naga Jagan Mohan Rao Thattukolla | S1 |
| D3 | `services/generation_service.py` (GPT-2) | Python Module | Naga Jagan Mohan Rao Thattukolla | S2 |
| D4 | `services/wiki_service.py` + `fact_check_service.py` | Python Module | Shaik Sumiya Zainab | S2 |
| D5 | `services/storage_service.py` (SQLite + JSON) | Python Module | Tejesh Velivela | S2 |
| D6 | FastAPI Application (`main.py` + routers) | Python Application | Shaik Sumiya Zainab | S3 |
| D7 | Streamlit Frontend (5 pages) | Python Application | Tejesh Velivela | S3 |
| D8 | Pytest Test Suite | Python Test Files | Satvika Tallam | S1–S4 |
| D9 | pytest-cov Coverage Report (≥ 80 %) | HTML/Terminal Report | Satvika Tallam | S4 |
| D10 | `Dockerfile.api` + `Dockerfile.streamlit` | Dockerfiles | Tejesh Velivela | S4 |
| D11 | `docker-compose.yml` | YAML | Tejesh Velivela | S4 |
| D12 | `README.md` (Technical Setup Guide) | Markdown | Tejesh Velivela | S4 |
| D13 | System Design Document | Markdown | Shaik Sumiya Zainab | S4 |
| D14 | API Documentation (Swagger + Postman) | JSON / Markdown | Shaik Sumiya Zainab | S4 |
| D15 | Project Planning Document (this file) | Markdown | All | S1 |
| D16 | Final Project Report (SmartBridge) | PDF / Markdown | All | S4 |

---

## 8. Definition of Done

A task or user story is considered **Done** only when **all** of the following criteria are satisfied:

### Per-Task Definition of Done

- [ ] Code is written and passes all related unit tests locally.
- [ ] Code passes linting checks (flake8 / ruff — zero errors, zero warnings).
- [ ] Code is committed to a feature branch with a clear, conventional commit message.
- [ ] A pull request has been opened and reviewed by at least one other team member.
- [ ] Pull request CI checks (lint + test) are green.
- [ ] Code is merged to `develop` and the GitHub issue/task is closed.
- [ ] Relevant documentation is updated (docstrings, README section if applicable).

### Per-Sprint Definition of Done

#### Sprint 1 ✅ Done When:
- [ ] BART service is deployed and returns coherent summaries for ≥ 5 test inputs.
- [ ] Data schema (JSON + SQLite DDL) is documented and committed.
- [ ] GitHub Actions CI pipeline runs successfully on every push to `develop`.
- [ ] Sprint 1 unit tests pass with ≥ 70 % coverage on Sprint 1 modules.
- [ ] Sprint review held; retrospective action items documented.

#### Sprint 2 ✅ Done When:
- [ ] GPT-2 service generates professional messages for at least 5 distinct contact profiles.
- [ ] Fact-check service returns `VERIFIED` / `UNVERIFIED` / `NOT_FOUND` for sample claims.
- [ ] SQLite storage correctly persists and retrieves contact records (CRUD verified).
- [ ] All Sprint 2 services have unit tests with ≥ 75 % coverage.
- [ ] Sprint review held; retrospective action items documented.

#### Sprint 3 ✅ Done When:
- [ ] All 7 FastAPI endpoints return correct HTTP status codes and response bodies.
- [ ] Streamlit app runs without errors and all 5 pages are functional.
- [ ] Full E2E scenario (add contact → summarize → generate message → fact-check → save) completes successfully.
- [ ] Swagger UI (`/docs`) is accessible and all endpoints are documented.
- [ ] Sprint review held; retrospective action items documented.

#### Sprint 4 ✅ Done When:
- [ ] pytest-cov report shows ≥ 80 % line coverage across all modules.
- [ ] `docker-compose up` successfully starts all services with zero errors.
- [ ] All project documents (README, System Design, API Docs, Planning Doc) are finalized and committed.
- [ ] Final Project Report is submitted to SmartBridge portal.
- [ ] Final demo is delivered and signed off by team lead.

---

## 9. Planning Tools

| Tool | Purpose | Usage in Project |
|------|---------|-----------------|
| **GitHub** | Source control & collaboration | All source code, pull requests, code reviews, and issue tracking. Branching strategy: `main` (stable) → `develop` (integration) → `feature/*` (individual tasks). |
| **GitHub Projects** | Agile project board | Kanban board with columns: Backlog → Sprint Backlog → In Progress → Review → Done. Each sprint's tasks are loaded as cards at the start of the sprint. |
| **GitHub Actions** | CI/CD automation | Workflows: (1) Lint on every push; (2) Run pytest on every push to `develop` or `main`; (3) Coverage gate — fail if coverage drops below threshold. |
| **SmartBridge Portal** | Academic project management & submission | Submit interim deliverables and final project report as per SmartBridge program requirements. Track compliance milestones. |
| **pytest** | Test framework | Unit, integration, and E2E test runner. Used with `pytest-cov` for coverage reporting and `pytest-mock` for mocking external dependencies. |
| **pytest-cov** | Code coverage | Generates line/branch coverage reports. Configured via `.coveragerc` to exclude test files, migrations, and `__init__.py` from coverage metrics. |
| **Docker / Docker Compose** | Containerization & deployment | Package and run the application as isolated containers. `docker-compose.yml` orchestrates the FastAPI and Streamlit services. |
| **Markdown** | Documentation | All project documentation written in Markdown for readability and GitHub rendering. |
| **VS Code** | IDE | Recommended IDE with Python, Pylance, and Docker extensions. Shared `.vscode/settings.json` committed for consistent formatting. |

---

## 10. Approval

This Project Planning Document has been reviewed and approved by the project team.

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Team Lead | Shaik Sumiya Zainab | ___________________ | July 7, 2026 |
| NLP Engineer | Naga Jagan Mohan Rao Thattukolla | ___________________ | July 7, 2026 |
| QA Lead | Satvika Tallam | ___________________ | July 7, 2026 |
| Frontend / DevOps | Tejesh Velivela | ___________________ | July 7, 2026 |

---

*Document prepared for the Personalized Networking Assistant project. Maintained by Shaik Sumiya Zainab (Team Lead). Last updated: July 7, 2026.*
