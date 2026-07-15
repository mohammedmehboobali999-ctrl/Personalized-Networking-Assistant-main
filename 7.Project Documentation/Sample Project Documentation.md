# Personalized Networking Assistant — Full Project Documentation

> **Version:** 1.0.0  
> **Date:** July 2026  
> **Repository:** [https://github.com/Sumiyazainab2308/Personalized-Networking-Assistant](https://github.com/Sumiyazainab2308/Personalized-Networking-Assistant)  
> **Status:** Completed

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Team](#2-team)
3. [Problem Statement](#3-problem-statement)
4. [Solution Overview](#4-solution-overview)
5. [Technology Stack](#5-technology-stack)
6. [System Architecture](#6-system-architecture)
7. [Service Layer](#7-service-layer)
8. [API Documentation](#8-api-documentation)
9. [Frontend Design](#9-frontend-design)
10. [Testing Strategy](#10-testing-strategy)
11. [Deployment](#11-deployment)
12. [Results & Screenshots](#12-results--screenshots)
13. [Challenges & Solutions](#13-challenges--solutions)
14. [Conclusion & Future Work](#14-conclusion--future-work)

---

## 1. Introduction

### 1.1 Project Overview

The **Personalized Networking Assistant** is an AI-powered web application designed to help professionals prepare for networking events, conferences, meetups, and industry summits. By combining the power of **BART** (a state-of-the-art transformer model for text summarisation), **GPT-2** (a generative language model for text generation), and the **Wikipedia REST API** (for real-time fact verification), the assistant transforms raw event descriptions into actionable, personalized networking intelligence.

Users provide minimal input — an event name, a description, and their professional profile — and the assistant returns:

- **Themed talking points** extracted from the event context
- **Personalized conversation starters** tailored to the user's role and industry
- **Fact-checked claims** with confidence scores and Wikipedia citation links

The application exposes its intelligence through a **FastAPI** backend (consumed programmatically or via Swagger) and a polished **Streamlit** web interface for non-technical end users.

### 1.2 Objectives

1. **Automate event analysis:** Extract meaningful themes and summaries from unstructured event descriptions using NLP.
2. **Personalize networking content:** Generate conversation topics tailored to the user's professional identity.
3. **Ensure factual accuracy:** Cross-reference generated content against Wikipedia to flag uncertain claims.
4. **Provide developer-friendly APIs:** Expose all AI capabilities as RESTful HTTP endpoints with full OpenAPI documentation.
5. **Enable effortless deployment:** Package the full stack in Docker for one-command deployment.
6. **Maintain code quality:** Achieve ≥ 80% test coverage across all service modules.

### 1.3 Scope

| In Scope | Out of Scope |
|----------|-------------|
| Event theme extraction (BART) | Real-time speaker/agenda integration |
| Personalized topic generation (GPT-2) | Calendar or event platform integrations |
| Wikipedia-based fact checking | User authentication & accounts |
| Session history persistence | Multi-language support (v1 is English only) |
| Streamlit web UI | Mobile native apps |
| FastAPI REST API | Paid/commercial AI API usage |
| Docker deployment | Cloud auto-scaling infrastructure |
| 38-test pytest suite | Load testing at scale |

---

## 2. Team

The Personalized Networking Assistant was designed, developed, and tested by a team of four engineers over the course of the project period.

| Name | Role | Responsibilities |
|------|------|-----------------|
| **mehboob ali mohammed** | Developer | Full-stack development, project architecture, API design, BART/GPT-2 integration, testing, documentation, DevOps |

### 2.1 Collaboration Tools

- **Version Control:** Git + GitHub
- **Issue Tracking:** GitHub Issues
- **Code Review:** GitHub Pull Requests (minimum 1 reviewer approval required)
- **Communication:** WhatsApp group + scheduled weekly sync calls
- **Documentation:** Markdown files in `/docs` and `/7.Project Documentation`

---

## 3. Problem Statement

### 3.1 The Networking Challenge

Professional networking is widely recognized as one of the most powerful tools for career advancement, business development, and knowledge sharing. Yet for many professionals — especially those new to an industry or attending a large, unfamiliar event — the experience is fraught with anxiety and inefficiency.

**Key pain points:**

1. **Lack of preparation:** Professionals arrive at events without researching relevant topics, resulting in shallow, forgettable conversations.
2. **Generic icebreakers:** Most people resort to the same overused openers ("So, what do you do?"), making it difficult to stand out.
3. **Information overload:** Event websites and brochures contain vast amounts of unstructured text that users lack the time to parse.
4. **Factual uncertainty:** Professionals risk embarrassing themselves by confidently stating claims that are inaccurate or outdated.
5. **No personalization:** Existing networking tools offer generic tips rather than advice tailored to the individual's role, industry, and the specific event.

### 3.2 The Gap in Existing Tools

Current networking preparation tools fall into two categories:
- **Generic advice articles** (e.g., "10 tips for networking") — not personalized, not event-specific
- **Manual research** — time-consuming, unstructured, cognitively demanding

No existing accessible tool combines **AI-driven event understanding**, **personalized topic generation**, and **factual verification** in a single, unified interface.

### 3.3 The Opportunity

With the advent of powerful, open-source pre-trained language models (BART, GPT-2) and freely accessible knowledge APIs (Wikipedia), it is now possible to build an intelligent assistant that bridges this gap — transforming raw event information into a personalized, verified networking preparation report in seconds.

---

## 4. Solution Overview

### 4.1 Core Concept

The Personalized Networking Assistant implements a **three-stage AI pipeline**:

```
Stage 1: UNDERSTAND           Stage 2: GENERATE           Stage 3: VERIFY
─────────────────────         ──────────────────────       ─────────────────────
Event Description (raw)  →→   Themes + Summary       →→   Topics + Claims
        ↓                           ↓                            ↓
  BART Summariser            GPT-2 Generator           Wikipedia Fact Checker
        ↓                           ↓                            ↓
  Themes, Talking Points     Personalized Topics        Confidence Score + Sources
```

### 4.2 User Journey

1. **User opens the Streamlit app** at `http://localhost:8501`
2. **Event Analyzer tab:** User enters event name, description, and industry → BART extracts themes and generates a structured summary
3. **Topic Generator tab:** User enters their name and professional role → GPT-2 generates 5 personalized conversation starters based on the event themes
4. **Fact Checker tab:** User enters any claim → Wikipedia API returns a confidence score and relevant source links
5. **History tab:** User reviews their previous sessions and generated content
6. **Feedback:** User rates the quality of the generated content

### 4.3 Key Differentiators

| Feature | Our Solution | Generic Tools |
|---------|-------------|---------------|
| Event-specific analysis | ✅ BART understands event context | ❌ Generic advice |
| Role-based personalization | ✅ GPT-2 generates user-specific topics | ❌ One-size-fits-all |
| Real-time fact verification | ✅ Wikipedia API cross-reference | ❌ No verification |
| Open-source, free models | ✅ BART + GPT-2 (no API costs) | ❌ Often requires paid APIs |
| REST API access | ✅ FastAPI with Swagger UI | ❌ Web-only interfaces |
| Docker deployment | ✅ One command setup | ❌ Complex manual setup |

---

## 5. Technology Stack

### 5.1 Full Stack Table

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| **Backend Framework** | FastAPI | 0.110.x | REST API, routing, validation, OpenAPI docs |
| **ASGI Server** | Uvicorn | 0.29.x | High-performance async HTTP server |
| **Frontend Framework** | Streamlit | 1.33.x | Interactive web UI (Python-native) |
| **AI — Summarisation** | BART (`facebook/bart-large-cnn`) | HuggingFace | Event theme extraction and summarisation |
| **AI — Generation** | GPT-2 (`gpt2`) | HuggingFace | Personalized topic and conversation generation |
| **NLP Library** | HuggingFace Transformers | 4.40.x | Model loading, inference pipelines |
| **Deep Learning** | PyTorch | 2.2.x | BART and GPT-2 inference backend |
| **Knowledge API** | Wikipedia REST API | v1 | Real-time fact verification and source retrieval |
| **Data Validation** | Pydantic | 2.x | Request/response schema validation |
| **HTTP Client** | Requests | 2.31.x | Wikipedia API and inter-service HTTP calls |
| **Testing** | pytest | 7.x | Unit and integration testing |
| **Mocking** | unittest.mock | stdlib | AI model mocking in tests |
| **Test Coverage** | pytest-cov | 4.x | Code coverage measurement |
| **Containerisation** | Docker + Docker Compose | 24.x + v2 | Application packaging and orchestration |
| **Language** | Python | 3.10.x | All application code |
| **Dependency Mgmt** | pip + requirements.txt | — | Package management |
| **Environment Config** | python-dotenv | 1.0.x | `.env` file loading |
| **Version Control** | Git + GitHub | — | Source control and collaboration |

### 5.2 AI Model Details

#### BART (Bidirectional and Auto-Regressive Transformer)

- **Model:** `facebook/bart-large-cnn`
- **Architecture:** Encoder-Decoder Transformer
- **Trained on:** CNN/DailyMail news summarisation dataset
- **Usage in project:** Summarises multi-paragraph event descriptions into concise themes and talking points
- **Input:** Raw event description text (up to 1024 tokens)
- **Output:** Structured summary (max 150 tokens)

#### GPT-2 (Generative Pre-trained Transformer 2)

- **Model:** `gpt2` (117M parameters)
- **Architecture:** Decoder-only Transformer
- **Trained on:** WebText (40 GB of internet text)
- **Usage in project:** Generates fluent, contextual conversation starter topics
- **Input:** Prompt containing event themes + user professional profile
- **Output:** 5 unique conversation topics (max 200 tokens each)

---

## 6. System Architecture

### 6.1 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        PRESENTATION TIER                             │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │              Streamlit Frontend (Port 8501)                   │   │
│   │   ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │   │
│   │   │ Event Analyzer│  │Topic Generator│  │  Fact Checker    │  │   │
│   │   │     Page      │  │     Page      │  │      Page        │  │   │
│   │   └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘  │   │
│   └──────────┼─────────────────┼───────────────────┼────────────┘   │
│              │  HTTP Requests  │                   │                 │
└──────────────┼─────────────────┼───────────────────┼─────────────────┘
               │                 │                   │
               ▼                 ▼                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         LOGIC / API TIER                             │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │              FastAPI Backend (Port 8000)                      │   │
│   │                                                              │   │
│   │  POST /analyze    POST /generate    POST /factcheck          │   │
│   │  GET  /history    POST /feedback    GET  /                   │   │
│   │                                    GET  /docs                │   │
│   │  ┌─────────────────────────────────────────────────────┐    │   │
│   │  │                   Service Layer                      │    │   │
│   │  │  ┌──────────────┐  ┌─────────────┐  ┌──────────┐   │    │   │
│   │  │  │ EventAnalyzer│  │TopicGenerator│  │FactChecker│  │    │   │
│   │  │  │   (BART)     │  │   (GPT-2)   │  │(Wikipedia)│  │    │   │
│   │  │  └──────────────┘  └─────────────┘  └──────────┘   │    │   │
│   │  │  ┌──────────────┐  ┌─────────────┐                  │    │   │
│   │  │  │HistoryManager│  │FeedbackHandler                  │    │   │
│   │  │  └──────────────┘  └─────────────┘                  │    │   │
│   │  └─────────────────────────────────────────────────────┘    │   │
│   └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
               │                 │                   │
               ▼                 ▼                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                           DATA TIER                                  │
│                                                                     │
│   ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐  │
│   │   HuggingFace   │   │   Wikipedia     │   │   Local JSON    │  │
│   │  Model Cache    │   │   REST API      │   │   Storage       │  │
│   │  (BART, GPT-2)  │   │  (External)     │   │(History/Feedback│  │
│   └─────────────────┘   └─────────────────┘   └─────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### 6.2 Data Flow

1. **User Input:** User submits data via Streamlit form
2. **HTTP Request:** Streamlit calls FastAPI endpoint via `requests.post()`
3. **Validation:** FastAPI + Pydantic validates the request body
4. **Service Invocation:** Router delegates to the appropriate service
5. **AI/API Call:** Service calls BART pipeline, GPT-2 pipeline, or Wikipedia API
6. **Response Assembly:** Service packages results into a structured dict
7. **JSON Response:** FastAPI serializes and returns the response
8. **UI Rendering:** Streamlit displays formatted results to the user
9. **Persistence:** Session data is appended to `session_history.json`

### 6.3 Component Communication

| From | To | Protocol | Format |
|------|----|----------|--------|
| Streamlit UI | FastAPI | HTTP/1.1 | JSON |
| FastAPI Router | Service | Python function call | Python dict |
| EventAnalyzer | BART | HuggingFace Pipeline | Python string |
| TopicGenerator | GPT-2 | HuggingFace Pipeline | Python string |
| FactChecker | Wikipedia API | HTTPS/REST | JSON |
| Services | File System | File I/O | JSON |

---

## 7. Service Layer

The service layer contains five service modules, each encapsulating a distinct domain of business logic.

### 7.1 `EventAnalyzer` — `app/services/event_analyzer.py`

**Model:** BART (`facebook/bart-large-cnn`)  
**Purpose:** Transforms raw, unstructured event descriptions into structured analytical output.

**Key Functions:**

| Function | Input | Output |
|----------|-------|--------|
| `analyze_event(event: dict)` | Event dict (name, description, industry) | `{themes, summary, talking_points}` |
| `extract_themes(text: str)` | Raw description string | `List[str]` of themes |
| `summarize_event(text: str)` | Raw description string | Summary string |
| `generate_talking_points(themes: List[str])` | List of themes | `List[str]` of talking points |

**Processing Logic:**

```python
def analyze_event(event: dict) -> dict:
    description = event.get("description", "")
    if not description:
        raise ValueError("Event description cannot be empty")
    
    # BART summarisation pipeline
    summary_result = summarizer(
        description,
        max_length=MAX_SUMMARY_LENGTH,
        min_length=30,
        do_sample=False
    )
    summary = summary_result[0]["summary_text"]
    
    # Extract themes from summary tokens
    themes = extract_themes(summary)
    
    # Generate talking points
    talking_points = generate_talking_points(themes)
    
    return {
        "event_name": event.get("event_name"),
        "industry": event.get("industry"),
        "summary": summary,
        "themes": themes,
        "talking_points": talking_points
    }
```

---

### 7.2 `TopicGenerator` — `app/services/topic_generator.py`

**Model:** GPT-2 (`gpt2`)  
**Purpose:** Generates personalized, contextually relevant conversation topics.

**Key Functions:**

| Function | Input | Output |
|----------|-------|--------|
| `generate_topics(themes, user_name, user_role, count)` | Themes list + user profile | `List[str]` of topics |
| `build_prompt(themes, user_name, user_role)` | Profile fields | Formatted prompt string |
| `deduplicate_topics(topics)` | Raw topic list | Deduplicated list |

**Prompt Engineering:**

The service constructs a carefully designed prompt to guide GPT-2 toward relevant, professional output:

```python
def build_prompt(themes: list, user_name: str, user_role: str) -> str:
    theme_str = ", ".join(themes[:5])  # Limit to top 5 themes
    return (
        f"As {user_name}, a {user_role}, I want to discuss the following topics "
        f"at a professional networking event focused on {theme_str}. "
        f"Here are 5 thoughtful conversation starters:\n1."
    )
```

---

### 7.3 `FactChecker` — `app/services/fact_checker.py`

**API:** Wikipedia REST API (`en.wikipedia.org/api/rest_v1/page/summary/{title}`)  
**Purpose:** Validates factual claims by cross-referencing Wikipedia content and returning a confidence score.

**Key Functions:**

| Function | Input | Output |
|----------|-------|--------|
| `check_fact(claim: str)` | Plain text claim | `{status, confidence, summary, sources}` |
| `extract_keyword(claim: str)` | Claim string | Most relevant keyword for Wikipedia lookup |
| `compute_confidence(claim, wiki_extract)` | Claim + Wikipedia text | Float (0.0–1.0) |
| `_fetch_wikipedia(title: str)` | Wikipedia title | Raw JSON response |

**Confidence Scoring Algorithm:**

The confidence score is computed as the **cosine similarity** (simplified using keyword overlap) between the user's claim and the Wikipedia extract:

```python
def compute_confidence(claim: str, wiki_extract: str) -> float:
    claim_words = set(claim.lower().split())
    wiki_words = set(wiki_extract.lower().split())
    
    if not claim_words or not wiki_words:
        return 0.0
    
    intersection = claim_words & wiki_words
    union = claim_words | wiki_words
    jaccard_similarity = len(intersection) / len(union)
    
    # Scale to 0–1 range with soft normalization
    return min(1.0, jaccard_similarity * 3.5)
```

**Error Handling:**

```python
try:
    response = requests.get(url, headers=headers, timeout=5)
    if response.status_code == 404:
        return {"status": "not_found", "confidence": 0.0, "sources": []}
    response.raise_for_status()
except requests.exceptions.Timeout:
    return {"status": "timeout", "confidence": 0.0, "message": "Wikipedia request timed out"}
except requests.exceptions.ConnectionError:
    return {"status": "error", "confidence": 0.0, "message": "Network connection failed"}
```

---

### 7.4 `HistoryManager` — `app/services/history_manager.py`

**Storage:** Local JSON file (`./data/session_history.json`)  
**Purpose:** Persists user session data across requests for audit trails and the history tab.

**Key Functions:**

| Function | Input | Output |
|----------|-------|--------|
| `save_session(session_data: dict)` | Session dict | None (writes to disk) |
| `get_all_sessions()` | — | `List[dict]` |
| `get_session(session_id: str)` | Session ID | `dict` or `None` |
| `clear_history()` | — | None |

---

### 7.5 `FeedbackHandler` — `app/services/feedback_handler.py`

**Storage:** Local JSON file (`./data/feedback.json`)  
**Purpose:** Collects, validates, and persists user feedback ratings and comments.

**Key Functions:**

| Function | Input | Output |
|----------|-------|--------|
| `save_feedback(feedback: dict)` | Feedback payload | `{"saved": True, "id": "..."}` |
| `get_all_feedback()` | — | `List[dict]` |
| `get_average_rating()` | — | `float` |

---

## 8. API Documentation

The FastAPI backend exposes **7 HTTP endpoints** documented at `/docs` (Swagger UI) and `/redoc`.

### 8.1 Endpoint Summary

| # | Method | Path | Service | Description |
|---|--------|------|---------|-------------|
| 1 | `GET` | `/` | — | Health check |
| 2 | `POST` | `/analyze` | EventAnalyzer | Analyze event and extract themes |
| 3 | `POST` | `/generate` | TopicGenerator | Generate personalized topics |
| 4 | `POST` | `/factcheck` | FactChecker | Verify a factual claim |
| 5 | `GET` | `/history` | HistoryManager | Get all session history |
| 6 | `POST` | `/feedback` | FeedbackHandler | Submit user feedback |
| 7 | `GET` | `/docs` | FastAPI | Swagger UI documentation |

---

### 8.2 `GET /` — Health Check

**Description:** Confirms the API server is running and responsive. Used by Docker health checks and monitoring tools.

**Response:**
```json
{
  "status": "ok",
  "version": "1.0.0",
  "message": "Personalized Networking Assistant API is running"
}
```

**Status Codes:** `200 OK`

---

### 8.3 `POST /analyze` — Analyze Event

**Description:** Accepts an event payload and returns AI-extracted themes, a summary, and talking points.

**Request Body:**
```json
{
  "event_name": "TechConnect 2026",
  "description": "A global summit bringing together 5,000+ AI and networking professionals to discuss the future of intelligent infrastructure, edge computing, and digital transformation strategies.",
  "industry": "Technology",
  "date": "2026-09-15"
}
```

**Response:**
```json
{
  "event_name": "TechConnect 2026",
  "industry": "Technology",
  "summary": "TechConnect 2026 is a major AI and networking summit focused on intelligent infrastructure, edge computing, and digital transformation.",
  "themes": ["Artificial Intelligence", "Edge Computing", "Digital Transformation", "Networking Infrastructure"],
  "talking_points": [
    "The convergence of AI and edge computing in modern infrastructure",
    "Digital transformation strategies for enterprise networking teams",
    "Open-source tooling for intelligent network management"
  ],
  "processing_time_ms": 3420
}
```

**Status Codes:** `200 OK` · `422 Unprocessable Entity` (validation error)

---

### 8.4 `POST /generate` — Generate Topics

**Description:** Accepts event themes and a user profile, returns personalized conversation starter topics.

**Request Body:**
```json
{
  "themes": ["Artificial Intelligence", "Edge Computing", "Digital Transformation"],
  "user_name": "Sumiya",
  "user_role": "ML Engineer",
  "count": 5
}
```

**Response:**
```json
{
  "user_name": "Sumiya",
  "user_role": "ML Engineer",
  "topics": [
    "How are you applying AI to optimize network traffic in your current projects?",
    "What challenges have you faced deploying ML models at the edge?",
    "Which open-source frameworks are you finding most effective for network intelligence?",
    "How do you see digital transformation reshaping MLOps practices over the next two years?",
    "What's your take on the role of synthetic data in training network anomaly detectors?"
  ],
  "processing_time_ms": 6850
}
```

**Status Codes:** `200 OK` · `422 Unprocessable Entity`

---

### 8.5 `POST /factcheck` — Fact Check

**Description:** Accepts a plain-text claim and returns a confidence score and Wikipedia sources.

**Request Body:**
```json
{
  "claim": "BART was developed by Facebook AI Research in 2019"
}
```

**Response:**
```json
{
  "claim": "BART was developed by Facebook AI Research in 2019",
  "status": "verified",
  "confidence": 0.82,
  "wikipedia_summary": "BART (Bidirectional and Auto-Regressive Transformers) is a denoising autoencoder for pretraining sequence-to-sequence models. It was introduced by Mike Lewis et al. at Facebook AI Research in 2019.",
  "sources": [
    {
      "title": "BART (language model)",
      "url": "https://en.wikipedia.org/wiki/BART_(language_model)"
    }
  ],
  "processing_time_ms": 1230
}
```

**Status Codes:** `200 OK` · `422 Unprocessable Entity`

---

### 8.6 `GET /history` — Get Session History

**Description:** Returns all stored session records from previous interactions.

**Response:**
```json
{
  "count": 3,
  "sessions": [
    {
      "session_id": "sess_20260715_001",
      "timestamp": "2026-07-15T10:23:45Z",
      "event_name": "TechConnect 2026",
      "themes": ["AI", "Edge Computing"],
      "topics_generated": 5
    }
  ]
}
```

**Status Codes:** `200 OK`

---

### 8.7 `POST /feedback` — Submit Feedback

**Description:** Accepts user feedback with a rating and optional comment.

**Request Body:**
```json
{
  "session_id": "sess_20260715_001",
  "rating": 5,
  "comment": "The topics were incredibly relevant and well-phrased. Very helpful!"
}
```

**Response:**
```json
{
  "saved": true,
  "feedback_id": "fb_20260715_001",
  "message": "Thank you for your feedback!"
}
```

**Status Codes:** `201 Created` · `422 Unprocessable Entity`

---

## 9. Frontend Design

### 9.1 Streamlit Application Overview

The frontend is a single-file Streamlit application (`frontend/streamlit_app.py`) organized into **four tabs** accessible from the top navigation:

```
┌──────────────────────────────────────────────────────────────────┐
│  🤝 Personalized Networking Assistant                             │
├──────────────────────────────────────────────────────────────────┤
│  [ 🎯 Event Analyzer ]  [ 💬 Topics ]  [ ✅ Fact Check ]  [ 📜 History ] │
└──────────────────────────────────────────────────────────────────┘
```

### 9.2 Page 1 — Event Analyzer Tab

**Purpose:** The starting point. User inputs event details and receives AI-generated analysis.

**UI Elements:**
- `st.text_input`: Event name field
- `st.text_area`: Event description (multi-line, min 3 rows)
- `st.selectbox`: Industry selection (Technology, Finance, Healthcare, Education, Other)
- `st.date_input`: Event date
- `st.button("Analyze Event")`: Triggers API call to `POST /analyze`

**Output Display:**
- `st.success`: Confirmation message
- `st.subheader("📌 Event Summary")` → `st.write(summary)`
- `st.subheader("🏷️ Key Themes")` → `st.columns` with theme badges
- `st.subheader("💡 Talking Points")` → numbered list with `st.markdown`

**State Management:**
```python
if "analysis_result" not in st.session_state:
    st.session_state.analysis_result = None
```

### 9.3 Page 2 — Topic Generator Tab

**Purpose:** Users provide their professional identity and receive personalized conversation starters.

**UI Elements:**
- `st.text_input`: Your name
- `st.text_input`: Your professional role
- `st.multiselect`: Select themes (populated from Event Analyzer results)
- `st.slider`: Number of topics (1–10, default 5)
- `st.button("Generate Topics")`: Calls `POST /generate`

**Output Display:**
- Numbered conversation topics in expandable `st.expander` blocks
- Copy button for each topic (clipboard integration)
- `st.download_button`: Export topics as `.txt`

### 9.4 Page 3 — Fact Checker Tab

**Purpose:** Allows users to verify any claim before stating it at an event.

**UI Elements:**
- `st.text_area`: Claim input
- `st.button("Check Fact")`: Calls `POST /factcheck`

**Output Display:**
- `st.metric`: Confidence score displayed as a percentage gauge
- Color-coded status badge:
  - 🟢 `verified` (confidence ≥ 0.7)
  - 🟡 `uncertain` (0.4 ≤ confidence < 0.7)
  - 🔴 `unverified` (confidence < 0.4)
- Wikipedia summary in `st.info` block
- Source links as `st.markdown("[Title](url)")`

### 9.5 Page 4 — History Tab

**Purpose:** Displays a log of all previous sessions in the current browser session.

**UI Elements:**
- `st.dataframe`: Table of past sessions with timestamps, event names, and topic counts
- `st.button("Clear History")`: Calls `DELETE /history`
- Per-row expand: shows full topic list for each past session

### 9.6 Sidebar

The sidebar provides:
- Application logo and title
- API connection status indicator (green ✅ / red ❌)
- Link to FastAPI Swagger docs (`http://localhost:8000/docs`)
- Feedback star rating (`st.feedback("stars")`)
- Team credits

---

## 10. Testing Strategy

### 10.1 Overview

The project uses **pytest** as its testing framework with **unittest.mock** for AI model isolation. The suite contains **38 tests** across 4 files.

| Test File | Module Tested | Tests | Type |
|-----------|--------------|-------|------|
| `test_event_analyzer.py` | `event_analyzer.py` | 10 | Unit |
| `test_topic_generator.py` | `topic_generator.py` | 10 | Unit |
| `test_fact_checker.py` | `fact_checker.py` | 12 | Unit |
| `test_api_routes.py` | FastAPI routes | 6 | Integration |
| **Total** | — | **38** | — |

### 10.2 Mocking Philosophy

All AI model pipelines (BART, GPT-2) and external HTTP calls (Wikipedia API) are replaced with `MagicMock` objects during testing. This ensures:

- ✅ Tests run in **< 5 seconds** (vs. 60+ seconds with real models)
- ✅ Tests are **deterministic** (no randomness from model sampling)
- ✅ Tests run **offline** (no internet required)
- ✅ Tests are **isolated** (failures are attributable to logic, not network)

```python
@patch("app.services.event_analyzer.summarizer")
def test_analyze_event_full_output_structure(mock_summarizer, sample_event):
    mock_summarizer.return_value = [{"summary_text": "AI networking summit."}]
    result = analyze_event(sample_event)
    assert all(k in result for k in ["themes", "summary", "talking_points"])
```

### 10.3 Edge Cases Covered

| Scenario | Test File | Outcome Verified |
|----------|-----------|-----------------|
| Empty description | `test_event_analyzer.py` | `ValueError` raised |
| Empty themes list | `test_topic_generator.py` | Defaults or `ValueError` |
| Long description (> 2000 chars) | `test_topic_generator.py` | Input truncated to 512 tokens |
| Empty claim | `test_fact_checker.py` | `ValueError` raised |
| Network failure | `test_fact_checker.py` | `{"status": "error"}` returned |
| Wikipedia 404 | `test_fact_checker.py` | `{"status": "not_found"}` |
| Request timeout | `test_fact_checker.py` | `{"status": "timeout"}` |
| Missing required fields | `test_api_routes.py` | HTTP `422` returned |
| Unicode / special characters | `test_event_analyzer.py` | No crash, handled cleanly |
| Duplicate topics in output | `test_topic_generator.py` | Deduplicated before return |

### 10.4 Code Coverage Results

```
TOTAL    387    39    90%   ← Exceeds 80% target ✅
```

### 10.5 Running Tests

```bash
# All tests
pytest tests/ -v

# With coverage
pytest tests/ -v --cov=app --cov-report=term-missing

# HTML report
pytest tests/ -v --cov=app --cov-report=html
```

---

## 11. Deployment

### 11.1 Local Development Deployment

**Requirements:** Python 3.10+, pip, virtualenv

```bash
# 1. Clone
git clone https://github.com/Sumiyazainab2308/Personalized-Networking-Assistant.git
cd Personalized-Networking-Assistant

# 2. Virtual environment
python -m venv venv && source venv/bin/activate

# 3. Dependencies
pip install -r requirements.txt

# 4. Configure
cp .env.example .env

# 5. Run API (Terminal 1)
uvicorn app.main:app --reload

# 6. Run Streamlit (Terminal 2)
streamlit run frontend/streamlit_app.py
```

### 11.2 Docker Deployment

**Requirements:** Docker Desktop, Docker Compose v2

```bash
# Build and start all services
docker compose up --build

# Background mode
docker compose up --build -d

# Stop
docker compose down
```

**Docker Architecture:**

| Container | Image Source | Port | Role |
|-----------|-------------|------|------|
| `networking-assistant-api` | `Dockerfile.api` | 8000 | FastAPI backend |
| `networking-assistant-frontend` | `Dockerfile.frontend` | 8501 | Streamlit frontend |

**Volume:** `model-cache` persists BART and GPT-2 weights between container restarts, avoiding repeated downloads.

### 11.3 Access URLs

| Interface | URL |
|-----------|-----|
| Streamlit UI | http://localhost:8501 |
| FastAPI Swagger | http://localhost:8000/docs |
| FastAPI ReDoc | http://localhost:8000/redoc |
| Health Check | http://localhost:8000/ |

### 11.4 Production Considerations

For production deployment beyond local or Docker:

1. **Use multiple Uvicorn workers:** `--workers 4` for CPU-bound concurrency
2. **Enable HTTPS:** Use a reverse proxy (Nginx, Traefik) with SSL/TLS
3. **GPU acceleration:** Mount a CUDA-capable GPU for 10–50× faster BART/GPT-2 inference
4. **Environment secrets:** Use Docker secrets or a secrets manager (AWS Secrets Manager, HashiCorp Vault) instead of `.env` files
5. **Health monitoring:** Integrate the `GET /` endpoint with uptime monitors (UptimeRobot, Grafana)
6. **Log aggregation:** Forward Uvicorn/Streamlit logs to a centralized logging service

---

## 12. Results & Screenshots

### 12.1 Event Analyzer — Output Example

**Input:**
- Event: "Global AI Ethics Summit 2026"
- Description: "A two-day summit gathering 3,000 AI researchers, ethicists, policy makers, and industry leaders to address bias in AI systems, regulatory frameworks, responsible AI deployment, and the societal impact of autonomous decision-making systems."
- Industry: Technology

**Output Themes:**
`AI Ethics` · `Algorithmic Bias` · `AI Regulation` · `Responsible AI` · `Autonomous Systems`

**Generated Summary:**
> "The Global AI Ethics Summit 2026 is a major gathering of researchers and industry professionals focused on addressing AI bias, establishing regulatory frameworks, and ensuring responsible deployment of autonomous AI systems."

**Talking Points:**
1. The role of transparency in reducing algorithmic bias in hiring and lending decisions
2. How the EU AI Act will reshape enterprise AI deployment strategies
3. Balancing innovation speed with responsible AI governance frameworks
4. Measuring societal impact of autonomous decision-making at scale

### 12.2 Topic Generator — Output Example

**User Profile:** Tejesh Velivela, AI Safety Researcher

**Generated Topics:**
1. "How are you approaching bias audits for large-scale production AI systems in your research?"
2. "What regulatory frameworks do you believe are most workable for practitioners — GDPR-style rules or principles-based guidelines?"
3. "How do you think about the tension between model explainability and performance in safety-critical systems?"
4. "What open datasets are you using for AI ethics benchmarking, and what gaps do you see?"
5. "How has the conversation around autonomous decision-making shifted in your community since the last year's regulatory changes?"

### 12.3 Fact Checker — Output Example

**Claim:** "GPT-2 was released by OpenAI in 2019"

**Result:**
- Status: ✅ Verified
- Confidence: **0.88** (88%)
- Wikipedia Summary: "GPT-2 is a large language model trained on a large corpus of English text. It was created by OpenAI and originally released in February 2019..."
- Source: [GPT-2 — Wikipedia](https://en.wikipedia.org/wiki/GPT-2)

### 12.4 Test Run Output

```
========================= test session starts ==========================
platform win32 -- Python 3.10.12, pytest-7.4.0, pluggy-1.3.0
collected 38 items

tests/test_event_analyzer.py::test_extract_themes_basic PASSED   [  2%]
...
tests/test_api_routes.py::test_submit_feedback PASSED            [100%]

========================= 38 passed in 3.42s ===========================

---------- coverage: 90% ----------
```

---

## 13. Challenges & Solutions

### 13.1 Challenge: Model Load Time

**Problem:** BART (`facebook/bart-large-cnn`) takes 45–60 seconds to load from disk on first invocation, causing unacceptable API response times for the first request.

**Solution:** Implement model pre-loading on FastAPI startup using the `@app.on_event("startup")` lifecycle hook. This loads both BART and GPT-2 once when the server starts, so all subsequent API calls use already-loaded in-memory models.

```python
@app.on_event("startup")
async def load_models():
    from app.services.event_analyzer import preload_bart
    from app.services.topic_generator import preload_gpt2
    preload_bart()
    preload_gpt2()
    logger.info("AI models loaded successfully.")
```

---

### 13.2 Challenge: Test Speed with Real AI Models

**Problem:** Running tests with real BART and GPT-2 models made the test suite take 5–10 minutes, unacceptable for rapid development iteration.

**Solution:** Adopted `unittest.mock.patch` to replace all AI pipeline calls with `MagicMock` objects. Tests now run in under 5 seconds. Mock return values are carefully designed to match real pipeline output shapes.

---

### 13.3 Challenge: GPT-2 Topic Quality

**Problem:** GPT-2's base model (117M parameters) generated off-topic, repetitive, or grammatically poor topics when given naive prompts.

**Solution:** Implemented structured **prompt engineering** with explicit role context, topic count hints, and numbered list formatting. Also added post-processing: output parsing, deduplication, and length filtering to ensure only high-quality topics reach the user.

---

### 13.4 Challenge: Wikipedia API Reliability

**Problem:** Wikipedia API calls sometimes returned 404 (topic not found), timed out, or returned empty extracts for obscure or newly created concepts.

**Solution:** Implemented a **multi-layered fallback strategy**:
1. Try exact keyword match against Wikipedia
2. On 404, try the most common synonyms/alternative phrasings
3. On network error/timeout, return graceful `{"status": "error"}` response
4. Cache successful lookups (in-memory `dict`) to avoid redundant API calls

---

### 13.5 Challenge: CORS Between FastAPI and Streamlit

**Problem:** Browser blocked Streamlit's cross-origin HTTP requests to FastAPI on a different port, resulting in `CORS policy` errors.

**Solution:** Added `CORSMiddleware` to FastAPI with explicit allowed origins configured via the `ALLOWED_ORIGINS` environment variable:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS.split(","),
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

### 13.6 Challenge: Docker Container Model Download

**Problem:** Every `docker compose up --build` re-downloaded BART model weights (~1.4 GB), making rebuilds slow and bandwidth-intensive.

**Solution:** Created a named Docker volume (`model-cache`) mounted at `/root/.cache` inside the container. HuggingFace's `transformers` library caches models at this path, so weights persist across container rebuilds.

---

## 14. Conclusion & Future Work

### 14.1 Conclusion

The Personalized Networking Assistant successfully demonstrates the practical application of state-of-the-art open-source AI models in a real-world productivity tool. By combining BART's extractive summarisation, GPT-2's generative capabilities, and Wikipedia's knowledge base, the project delivers measurable value: transforming unstructured event information into personalized, fact-verified networking intelligence within seconds.

**Key achievements:**
- ✅ Full-stack AI application built with FastAPI + Streamlit
- ✅ Three AI capabilities integrated: summarisation, generation, fact-checking
- ✅ 38-test suite with 90% code coverage (exceeding 80% target)
- ✅ Docker deployment with one-command setup
- ✅ Professional API documentation with Swagger UI
- ✅ Clean, modular service architecture with clear separation of concerns

The project demonstrates that powerful AI-driven applications can be built entirely with open-source, cost-free models — making advanced NLP accessible without reliance on paid API services.

### 14.2 Future Work

| Priority | Enhancement | Description |
|----------|-------------|-------------|
| 🔴 High | **Speaker/agenda integration** | Scrape or import event agendas to generate more specific, speaker-referenced topics |
| 🔴 High | **User authentication** | Add JWT-based auth so users can save and retrieve their personalized history across sessions |
| 🟡 Medium | **Better language models** | Upgrade from GPT-2 (117M) to Llama 3 or Mistral 7B for significantly higher quality topic generation |
| 🟡 Medium | **Multi-language support** | Extend BART and Wikipedia integration to support Spanish, French, German, and other languages |
| 🟡 Medium | **LinkedIn integration** | Allow users to import their LinkedIn profile for richer personalization of topics |
| 🟡 Medium | **Confidence calibration** | Replace the Jaccard-based confidence score with a proper semantic similarity model (e.g., Sentence-BERT) |
| 🟢 Low | **Mobile-responsive UI** | Migrate frontend to a React/Vue app with a mobile-first design |
| 🟢 Low | **Real-time collaboration** | Add WebSocket support for multiple users to co-prepare for the same event |
| 🟢 Low | **Analytics dashboard** | Admin dashboard showing usage statistics, popular events, and average confidence scores |
| 🟢 Low | **Kubernetes deployment** | Helm chart for production-scale Kubernetes deployment with auto-scaling |

### 14.3 Lessons Learned

1. **Prompt engineering matters enormously** for small generative models like GPT-2 — investing time in prompt design yields outsized quality improvements.
2. **Mock early, mock thoroughly** — integrating `unittest.mock` from day one kept the test suite fast and maintainable throughout the project.
3. **Docker volume strategy** for ML model caches is essential — rebuild times drop from 15+ minutes to under 2 minutes with proper volume configuration.
4. **API-first design** using FastAPI + Pydantic schemas catches contract mismatches between frontend and backend before they become runtime bugs.
5. **Separation of concerns** across 5 service modules made the codebase maintainable and made individual contributions by team members non-conflicting.

---

## References

1. Lewis, M., et al. (2019). *BART: Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension*. Facebook AI Research. [https://arxiv.org/abs/1910.13461](https://arxiv.org/abs/1910.13461)
2. Radford, A., et al. (2019). *Language Models are Unsupervised Multitask Learners*. OpenAI. [https://openai.com/research/gpt-2](https://openai.com/research/gpt-2)
3. Wikipedia REST API Documentation. [https://en.wikipedia.org/api/rest_v1/](https://en.wikipedia.org/api/rest_v1/)
4. FastAPI Documentation. [https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)
5. Streamlit Documentation. [https://docs.streamlit.io/](https://docs.streamlit.io/)
6. HuggingFace Transformers Documentation. [https://huggingface.co/docs/transformers](https://huggingface.co/docs/transformers)
7. pytest Documentation. [https://docs.pytest.org/en/stable/](https://docs.pytest.org/en/stable/)
8. Docker Compose Documentation. [https://docs.docker.com/compose/](https://docs.docker.com/compose/)

---

*Personalized Networking Assistant — Full Project Documentation*  
*Team Lead: **Shaik Sumiya Zainab***  
*Members: **Naga Jagan Mohan Rao Thattukolla** · **Satvika Tallam** · **Tejesh Velivela***  
*July 2026*
