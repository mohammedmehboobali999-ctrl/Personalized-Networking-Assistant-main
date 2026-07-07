# No. of Functional Features Included in the Solution
## Personalized Networking Assistant

**Project:** Personalized Networking Assistant  
**Team Lead:** Shaik Sumiya Zainab  
**Team Members:** Naga Jagan Mohan Rao Thattukolla, Satvika Tallam, Tejesh Velivela  
**Version:** 1.0  
**Date:** July 2026  

---

## 1. Overview & Feature Summary

The **Personalized Networking Assistant** incorporates **12 distinct functional features** across its presentation, API, and machine learning layers. Each feature has been designed to address specific user pain points identified during the requirement analysis and empathy mapping phases. 

Out of the 12 functional features defined in the project specification, **100% (12/12) are fully implemented and verified** in the production codebase.

---

## 2. Complete Functional Feature Inventory

| Feature ID | Feature Name | Core Description | Primary Technologies & Models | Status |
|---|---|---|---|---|
| **F-01** | **Event Theme Extraction** | Analyzes raw event descriptions and extracts the top 3 most relevant professional themes using zero-shot NLP classification. | HuggingFace BART (`facebook/bart-large-mnli`), FastAPI | ✅ Implemented |
| **F-02** | **Starter Generation** | Synthesizes 2 to 5 tailored, context-aware networking conversation starters based on extracted themes and user bio. | HuggingFace GPT-2 Small (`gpt2`), Prompt Engineering | ✅ Implemented |
| **F-03** | **Wikipedia Fact-Checking** | Verifies claims and retrieves instant background summaries for technical or industry topics before networking events. | Wikipedia REST API, `requests`, Custom Fallback Handler | ✅ Implemented |
| **F-04** | **Session History Logging** | Automatically logs every generated conversation session with timestamps, themes, and prompts to persistent storage. | `pathlib`, Atomic JSON I/O (`data/history.json`) | ✅ Implemented |
| **F-05** | **Interactive Feedback System**| Allows users to rate individual starters with 👍/👎 buttons and submit overall 1–5 star ratings and comments. | Streamlit Session State, Atomic JSON I/O (`data/feedback.json`)| ✅ Implemented |
| **F-06** | **Paginated History Review** | Enables users to browse past networking sessions in reverse-chronological order with keyword filtering and pagination. | FastAPI Pagination (`page`, `page_size`), Streamlit Expanders| ✅ Implemented |
| **F-07** | **Feedback History View** | Displays the 10 most recently rated conversation starters with intuitive emoji indicators and exact timestamps. | `feedback_logger.py`, Streamlit Data Presentation | ✅ Implemented |
| **F-08** | **Analytics Dashboard** | Visualizes aggregate user metrics including total sessions generated, average ratings, and most frequent networking themes. | Streamlit Metrics, Dynamic Bar Charts & Frequency Distribution | ✅ Implemented |
| **F-09** | **Tabbed Web Interface** | Provides a seamless, dark-themed responsive user interface organized into intuitive functional tabs and sidebar navigation.| Streamlit 1.35+, Custom Layout Columns & Containers | ✅ Implemented |
| **F-10** | **OpenAPI / Swagger Docs** | Automatically generates interactive API documentation allowing developers to test endpoints directly from the browser. | FastAPI OpenAPI Generator, Swagger UI (`/docs`), ReDoc (`/redoc`)| ✅ Implemented |
| **F-11** | **Docker Containerization** | Encapsulates backend API and frontend web UI into isolated, reproducible containers orchestrated via Docker Compose. | Docker, Docker Compose, Multi-stage Builds | ✅ Implemented |
| **F-12** | **Starter Text Export** | Allows users to download their generated conversation starters and event summary as a formatted plain text file. | Streamlit Download Button (`st.download_button`) | ✅ Implemented |

---

## 3. Detailed Feature Specifications

### F-01: Event Theme Extraction
- **Module:** `app/services/nlp_service.py` & `app/services/event_analyzer.py`
- **Endpoint:** `POST /api/v1/analyze-event`
- **Technical Details:** Utilizes the `facebook/bart-large-mnli` model loaded via a thread-safe lazy singleton pattern. Evaluates candidate labels across technology, business, sustainability, healthcare, and finance domains. Scores multi-label entailment probabilities and returns the top 3 highest-ranking thematic categories in under 5 seconds.

### F-02: Conversation Starter Generation
- **Module:** `app/services/generation_service.py` & `app/services/topic_generator.py`
- **Endpoint:** `POST /api/v1/generate-conversation`
- **Technical Details:** Utilizes `gpt2` (124M parameters) on CPU. Constructs a structured prompt merging the extracted event themes with the user's professional bio or interests. Enforces post-processing filters to strip bullet markers, remove blank lines, and eliminate incomplete sentences, delivering clean, natural language openers.

### F-03: Wikipedia Fact-Checking
- **Module:** `app/services/fact_checker.py`
- **Endpoint:** `GET /api/v1/fact-check?query={topic}`
- **Technical Details:** Makes direct HTTP calls to `https://en.wikipedia.org/api/rest_v1/page/summary/{query}` with customized User-Agent headers. Returns structured data including article title, summary extract, canonical desktop URL, and confidence scoring (`high`, `medium`, `low`). Incorporates defensive exception handling to guarantee the app never crashes during network timeouts or 404 Not Found errors.

### F-04: Session History Logging
- **Module:** `app/services/storage_service.py` & `app/services/history_logger.py`
- **Endpoint:** Implicitly invoked on generation; retrieved via `GET /api/v1/history`
- **Technical Details:** Implements an atomic file-writing mechanism against `data/history.json`. Each session is assigned a unique UUIDv4 and stamped with an ISO-8601 UTC timestamp. Guarantees zero data loss or file corruption during rapid successive generations.

### F-05: Interactive Feedback System
- **Module:** `app/services/feedback_logger.py` & `frontend/streamlit_app.py`
- **Endpoint:** `POST /api/v1/feedback`
- **Technical Details:** Streamlit UI dynamically generates unique button keys for every starter card (`like_{idx}`, `dislike_{idx}`). Clicking 👍 or 👎 triggers an asynchronous payload to the backend, storing the suggestion text, action, star rating, and optional user feedback commentary in `data/feedback.json`.

### F-06: Paginated History Review
- **Module:** `app/routes/api.py` & `frontend/pages/history.py`
- **Endpoint:** `GET /api/v1/history?page=1&page_size=5&search=keyword`
- **Technical Details:** Backend slices history arrays based on pagination parameters and performs case-insensitive substring matching across event descriptions and generated starters. Streamlit renders results inside collapsible `st.expander` components for clean visual organization.

### F-07: Feedback History View
- **Module:** `app/services/feedback_logger.py` & `frontend/pages/history.py`
- **Endpoint:** `GET /api/v1/feedback?limit=10`
- **Technical Details:** Fetches the 10 most recent feedback records from disk. Renders a formatted timeline showing whether the user found the starter helpful (👍) or unhelpful (👎), along with the exact timestamp of evaluation.

### F-08: Analytics Dashboard
- **Module:** `frontend/pages/dashboard.py`
- **Endpoint:** Aggregates data from `/api/v1/history` and `/api/v1/feedback`
- **Technical Details:** Computes real-time key performance indicators (KPIs) using Streamlit metric cards. Extracts theme frequency distributions and renders interactive visual bar charts showing the most common networking topics explored by the user.

### F-09: Tabbed Web Interface
- **Module:** `frontend/streamlit_app.py`
- **Technical Details:** Designed with a professional dark aesthetic. Uses `st.tabs` to separate **Generate Starters**, **Fact-Check**, and **History & Feedback** workflows into distinct visual workspaces, preventing cognitive overload and eliminating page reload friction.

### F-10: OpenAPI / Swagger API Documentation
- **Module:** `app/main.py`
- **Endpoint:** `GET /docs` (Swagger UI) & `GET /redoc` (ReDoc)
- **Technical Details:** Leverages FastAPI's native OpenAPI integration. Provides complete JSON schemas, parameter descriptions, request body examples, and interactive "Try it out" functionality for all API endpoints.

### F-11: Docker Containerization
- **Module:** `Dockerfile`, `Dockerfile.frontend`, `docker-compose.yml`
- **Technical Details:** Establishes a dual-container architecture. Builds lightweight Python 3.11 images, installs locked dependencies, exposes ports 8000 (API) and 8501 (UI), and sets up shared volume mounts for persistent JSON data storage across container restarts.

### F-12: Starter Text Export
- **Module:** `frontend/streamlit_app.py`
- **Technical Details:** Generates a dynamically formatted text buffer containing the event description, extracted themes, and numbered conversation starters. Offers an instant one-click download via `st.download_button` so users can export openers to their mobile devices before attending networking events.

---

## 4. Feature Mapping to SmartBridge Epics

| SmartBridge Epic | Associated Functional Features | Verification Method |
|---|---|---|
| **Epic 1: Project Setup & Prereqs** | F-09 (UI Setup), F-10 (API Setup), F-11 (Docker) | `make run-api`, `docker compose up` |
| **Epic 2: NLP Theme Extraction** | F-01 (Theme Extraction) | `pytest tests/test_event_analyzer.py` |
| **Epic 3: GPT-2 Generation** | F-02 (Starter Generation), F-12 (Export) | `pytest tests/test_topic_generator.py` |
| **Epic 4: Wikipedia Fact-Checking**| F-03 (Wikipedia Verification) | `pytest tests/test_fact_checker.py` |
| **Epic 5: Storage & History** | F-04 (History Logging), F-06 (History Review) | `pytest tests/test_api.py -k history` |
| **Epic 6: Feedback & Analytics** | F-05 (Feedback System), F-07 (Feedback View), F-08 (Dashboard) | `pytest tests/test_api.py -k feedback` |
