# Performance Testing — Personalized Networking Assistant

> **Project:** Personalized Networking Assistant  
> **Framework:** pytest · unittest.mock  
> **Total Tests:** 38 (Unit + Integration)  
> **Coverage Target:** ≥ 80%  
> **Last Updated:** july 2026

---

## Table of Contents

1. [Testing Overview](#1-testing-overview)
2. [Test Environment Setup](#2-test-environment-setup)
3. [Test Categories](#3-test-categories)
   - 3.1 [Unit Tests — event_analyzer](#31-unit-tests--event_analyzer)
   - 3.2 [Unit Tests — topic_generator](#32-unit-tests--topic_generator)
   - 3.3 [Unit Tests — fact_checker](#33-unit-tests--fact_checker)
   - 3.4 [Integration Tests — API Routes](#34-integration-tests--api-routes)
4. [Mock Strategy for AI Models](#4-mock-strategy-for-ai-models)
5. [Performance Benchmarks](#5-performance-benchmarks)
6. [Edge Cases Tested](#6-edge-cases-tested)
7. [Test Results Summary](#7-test-results-summary)
8. [Code Coverage Report](#8-code-coverage-report)
9. [Running the Tests](#9-running-the-tests)

---

## 1. Testing Overview

The Personalized Networking Assistant employs a rigorous, multi-layered testing strategy built around **pytest**, Python's industry-standard testing framework. The test suite is designed to validate correctness, performance, and resilience of every component in the system — from individual AI service functions all the way through to HTTP API endpoints.

| Attribute | Value |
|-----------|-------|
| Testing Framework | `pytest 7.x` |
| Mocking Library | `unittest.mock` (stdlib) |
| HTTP Test Client | `FastAPI TestClient` (wraps `httpx`) |
| Coverage Tool | `pytest-cov` |
| Total Test Files | 4 |
| Total Test Cases | **38** |
| Unit Tests | 32 |
| Integration Tests | 6 |
| Coverage Target | **≥ 80%** |

### Test Suite Layout

```
tests/
├── __init__.py
├── test_event_analyzer.py       # 10 unit tests
├── test_topic_generator.py      # 10 unit tests
├── test_fact_checker.py         # 12 unit tests
└── test_api_routes.py           # 6 integration tests
```

---

## 2. Test Environment Setup

### 2.1 Prerequisites

Ensure the virtual environment is active and all dependencies — including test extras — are installed:

```bash
# Create and activate virtual environment
python -m venv venv
source venv/bin/activate        # Linux/macOS
venv\Scripts\activate           # Windows PowerShell

# Install all dependencies including test extras
pip install -r requirements.txt
pip install pytest pytest-cov pytest-asyncio httpx
```

### 2.2 Environment Variables for Testing

Create a `.env.test` file (or set the following in your shell) so tests never hit real external APIs:

```env
WIKIPEDIA_API_URL=http://mock-wikipedia.local
OPENAI_API_KEY=test-key-not-used
APP_ENV=testing
LOG_LEVEL=WARNING
```

### 2.3 conftest.py

A shared `conftest.py` provides reusable fixtures across all test modules:

```python
# tests/conftest.py
import pytest
from fastapi.testclient import TestClient
from unittest.mock import MagicMock, patch
from app.main import app

@pytest.fixture(scope="module")
def client():
    """FastAPI test client — reused across all integration tests."""
    with TestClient(app) as c:
        yield c

@pytest.fixture
def mock_bart_model():
    """Returns a MagicMock that mimics the BART summarisation pipeline."""
    mock = MagicMock()
    mock.return_value = [{"summary_text": "Mocked summary output"}]
    return mock

@pytest.fixture
def mock_gpt2_model():
    """Returns a MagicMock that mimics the GPT-2 text-generation pipeline."""
    mock = MagicMock()
    mock.return_value = [{"generated_text": "Mocked topic: AI in Networking"}]
    return mock

@pytest.fixture
def sample_event():
    return {
        "event_name": "TechConnect 2026",
        "description": "A global summit on AI and networking innovations.",
        "industry": "Technology",
        "date": "2026-09-15"
    }
```

---

## 3. Test Categories

### 3.1 Unit Tests — `event_analyzer`

**File:** `tests/test_event_analyzer.py`  
**Module Under Test:** `app/services/event_analyzer.py`  
**Total Tests:** 10

The `event_analyzer` service uses the **BART** transformer model (via HuggingFace `transformers`) to extract themes, summaries, and talking points from event descriptions. All model calls are mocked so tests remain fast and deterministic.

#### Test Cases

| # | Test Name | Description |
|---|-----------|-------------|
| 1 | `test_extract_themes_basic` | Verifies that theme extraction returns a non-empty list for a standard event description. |
| 2 | `test_extract_themes_returns_list` | Asserts the return type is `list`. |
| 3 | `test_extract_themes_with_short_input` | Confirms graceful handling of a very short (< 20 char) description. |
| 4 | `test_extract_themes_model_called_once` | Ensures the BART pipeline is invoked exactly once per request (no redundant calls). |
| 5 | `test_summarize_event_basic` | Validates that a summary string is returned for a valid event dict. |
| 6 | `test_summarize_event_length` | Checks the summary does not exceed `max_length` tokens set in config. |
| 7 | `test_summarize_event_empty_description` | Expects a `ValueError` or sensible default when description is empty string. |
| 8 | `test_analyze_event_full_output_structure` | Asserts the full output dict contains `themes`, `summary`, and `talking_points` keys. |
| 9 | `test_analyze_event_industry_tagging` | Checks that the `industry` field from input appears in the structured output. |
| 10 | `test_analyze_event_with_special_characters` | Tests that HTML entities, unicode, and special chars do not crash the analyzer. |

#### Sample Test (pytest style)

```python
# tests/test_event_analyzer.py
import pytest
from unittest.mock import patch, MagicMock
from app.services.event_analyzer import analyze_event

@patch("app.services.event_analyzer.summarizer")
def test_analyze_event_full_output_structure(mock_summarizer, sample_event):
    mock_summarizer.return_value = [{"summary_text": "AI networking summit themes."}]
    result = analyze_event(sample_event)
    assert "themes" in result
    assert "summary" in result
    assert "talking_points" in result

@patch("app.services.event_analyzer.summarizer")
def test_extract_themes_model_called_once(mock_summarizer, sample_event):
    mock_summarizer.return_value = [{"summary_text": "theme1, theme2"}]
    analyze_event(sample_event)
    mock_summarizer.assert_called_once()
```

---

### 3.2 Unit Tests — `topic_generator`

**File:** `tests/test_topic_generator.py`  
**Module Under Test:** `app/services/topic_generator.py`  
**Total Tests:** 10

The `topic_generator` service uses the **GPT-2** language model to produce personalized conversation topics and networking icebreakers based on user profile and event themes.

#### Test Cases

| # | Test Name | Description |
|---|-----------|-------------|
| 1 | `test_generate_topics_returns_list` | Asserts the return value is a list of strings. |
| 2 | `test_generate_topics_count` | Verifies exactly `n` topics are returned when `count=n` is specified. |
| 3 | `test_generate_topics_non_empty_strings` | Ensures no empty string topics are returned in the list. |
| 4 | `test_generate_topics_model_called` | Confirms GPT-2 pipeline is invoked during topic generation. |
| 5 | `test_generate_topics_with_empty_themes` | Checks behavior when the themes list is empty (returns defaults or raises). |
| 6 | `test_generate_topics_with_profile` | Validates that user profile fields (name, role) appear in prompt context. |
| 7 | `test_generate_topics_max_length` | Ensures each generated topic does not exceed `max_topic_length` characters. |
| 8 | `test_generate_topics_uniqueness` | Checks that all returned topics are unique (no duplicates). |
| 9 | `test_generate_topics_format` | Verifies topics are plain strings (no JSON artifacts or control chars). |
| 10 | `test_generate_topics_with_long_description` | Tests that very long input descriptions are truncated to model token limits. |

#### Sample Test

```python
# tests/test_topic_generator.py
import pytest
from unittest.mock import patch
from app.services.topic_generator import generate_topics

@patch("app.services.topic_generator.generator")
def test_generate_topics_count(mock_generator):
    mock_generator.return_value = [
        {"generated_text": f"Topic {i}"} for i in range(5)
    ]
    topics = generate_topics(themes=["AI", "Networking"], count=5)
    assert len(topics) == 5

@patch("app.services.topic_generator.generator")
def test_generate_topics_uniqueness(mock_generator):
    mock_generator.return_value = [
        {"generated_text": "How is AI reshaping enterprise networking?"},
        {"generated_text": "What open-source tools do you rely on?"},
    ]
    topics = generate_topics(themes=["AI"], count=2)
    assert len(topics) == len(set(topics))
```

---

### 3.3 Unit Tests — `fact_checker`

**File:** `tests/test_fact_checker.py`  
**Module Under Test:** `app/services/fact_checker.py`  
**Total Tests:** 12

The `fact_checker` service queries the **Wikipedia REST API** to validate factual claims extracted from generated topics and event descriptions. Responses are parsed, cross-referenced, and scored for confidence.

#### Test Cases

| # | Test Name | Description |
|---|-----------|-------------|
| 1 | `test_fact_check_returns_dict` | Validates the return type is a dict with expected keys. |
| 2 | `test_fact_check_confidence_range` | Ensures confidence score is a float between 0.0 and 1.0. |
| 3 | `test_fact_check_known_fact` | Checks a well-known fact ("Python is a programming language") returns high confidence. |
| 4 | `test_fact_check_unknown_claim` | Checks a nonsense claim returns low confidence or "unverified" status. |
| 5 | `test_fact_check_wikipedia_called` | Asserts the Wikipedia API is called during fact-checking. |
| 6 | `test_fact_check_wikipedia_not_called_cache_hit` | Verifies repeated same-query checks hit the cache and skip the API call. |
| 7 | `test_fact_check_network_failure` | Mocks a `requests.exceptions.ConnectionError` and checks graceful error handling. |
| 8 | `test_fact_check_404_response` | Mocks a 404 HTTP response from Wikipedia and checks the fallback behavior. |
| 9 | `test_fact_check_timeout` | Mocks a `requests.exceptions.Timeout` and confirms a timeout-handling code path. |
| 10 | `test_fact_check_empty_claim` | Submits empty string claim and expects a `ValueError`. |
| 11 | `test_fact_check_long_claim` | Validates truncation/handling of claims exceeding 500 characters. |
| 12 | `test_fact_check_sources_list` | Confirms the response contains a `sources` list with at least one Wikipedia URL. |

#### Sample Test

```python
# tests/test_fact_checker.py
import pytest
import requests
from unittest.mock import patch, MagicMock
from app.services.fact_checker import check_fact

@patch("app.services.fact_checker.requests.get")
def test_fact_check_network_failure(mock_get):
    mock_get.side_effect = requests.exceptions.ConnectionError("Network down")
    result = check_fact("AI was invented in 1956")
    assert result["status"] == "error"
    assert "network" in result["message"].lower()

@patch("app.services.fact_checker.requests.get")
def test_fact_check_404_response(mock_get):
    mock_response = MagicMock()
    mock_response.status_code = 404
    mock_get.return_value = mock_response
    result = check_fact("Nonexistent concept XYZ")
    assert result["status"] in ("unverified", "not_found")

@patch("app.services.fact_checker.requests.get")
def test_fact_check_confidence_range(mock_get):
    mock_response = MagicMock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"extract": "Python is a high-level programming language."}
    mock_get.return_value = mock_response
    result = check_fact("Python is a programming language")
    assert 0.0 <= result["confidence"] <= 1.0
```

---

### 3.4 Integration Tests — API Routes

**File:** `tests/test_api_routes.py`  
**Client:** `FastAPI TestClient`  
**Total Tests:** 6

Integration tests spin up the full FastAPI application (with mocked AI backends) and exercise each endpoint end-to-end: routing, request validation, response schema, and HTTP status codes.

#### Endpoints Tested

| # | Test Name | Endpoint | Method | Description |
|---|-----------|----------|--------|-------------|
| 1 | `test_health_check` | `GET /` | GET | Confirms the health check returns `200 OK` and `{"status": "ok"}`. |
| 2 | `test_analyze_event` | `POST /analyze` | POST | Submits an event payload and checks themes + summary are returned. |
| 3 | `test_generate_topics` | `POST /generate` | POST | Submits themes + profile and checks topics list is returned. |
| 4 | `test_fact_check` | `POST /factcheck` | POST | Submits a claim and checks confidence score and sources are returned. |
| 5 | `test_get_history` | `GET /history` | GET | Verifies the history endpoint returns a list (can be empty). |
| 6 | `test_submit_feedback` | `POST /feedback` | POST | Submits feedback payload and expects `201 Created` with acknowledgement. |

#### Sample Integration Tests

```python
# tests/test_api_routes.py
import pytest
from unittest.mock import patch
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_health_check():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json()["status"] == "ok"

@patch("app.services.event_analyzer.summarizer")
def test_analyze_event(mock_summarizer):
    mock_summarizer.return_value = [{"summary_text": "AI and Networking summit"}]
    payload = {
        "event_name": "TechConnect 2026",
        "description": "Global AI and networking summit for professionals.",
        "industry": "Technology"
    }
    response = client.post("/analyze", json=payload)
    assert response.status_code == 200
    data = response.json()
    assert "themes" in data
    assert "summary" in data

@patch("app.services.topic_generator.generator")
def test_generate_topics(mock_generator):
    mock_generator.return_value = [{"generated_text": "How is AI changing networking?"}]
    payload = {
        "themes": ["AI", "Networking"],
        "user_name": "Sumiya",
        "user_role": "ML Engineer"
    }
    response = client.post("/generate", json=payload)
    assert response.status_code == 200
    assert isinstance(response.json()["topics"], list)

def test_submit_feedback():
    payload = {"session_id": "abc123", "rating": 5, "comment": "Very helpful!"}
    response = client.post("/feedback", json=payload)
    assert response.status_code == 201
```

---

## 4. Mock Strategy for AI Models

Because BART and GPT-2 are large transformer models (hundreds of MB), loading them during every test run would make tests impractically slow and non-deterministic. The project uses **`unittest.mock.patch`** as a decorator or context manager to replace model pipeline objects with lightweight `MagicMock` instances.

### 4.1 Patching Philosophy

```
Real Module                        Test Module
──────────────────────────────     ──────────────────────────────────────────
app/services/event_analyzer.py     tests/test_event_analyzer.py
  summarizer = pipeline(...)    ←  @patch("app.services.event_analyzer.summarizer")
  
app/services/topic_generator.py    tests/test_topic_generator.py
  generator = pipeline(...)     ←  @patch("app.services.topic_generator.generator")

app/services/fact_checker.py       tests/test_fact_checker.py
  requests.get(...)             ←  @patch("app.services.fact_checker.requests.get")
```

### 4.2 Key Mocking Rules

1. **Patch at the point of use**, not at the point of definition. Always patch `app.services.<module>.<name>`, not `transformers.pipeline`.
2. **Return realistic shapes**: mocks must return objects that match the real pipeline output structure (e.g., `[{"summary_text": "..."}]`).
3. **Scope mocks tightly**: use `@patch` as a decorator per test rather than module-level patches to avoid state leakage.
4. **Verify calls**: use `mock.assert_called_once()`, `mock.assert_called_with(...)` to confirm the service logic is wiring correctly.

### 4.3 Example: Comprehensive Mock Setup

```python
from unittest.mock import patch, MagicMock

@patch("app.services.event_analyzer.summarizer")
@patch("app.services.topic_generator.generator")
def test_full_pipeline(mock_gen, mock_sum, sample_event):
    # Configure BART mock
    mock_sum.return_value = [{"summary_text": "AI summit themes extracted."}]
    # Configure GPT-2 mock
    mock_gen.return_value = [{"generated_text": "Topic: Future of AI Networking"}]

    from app.services.event_analyzer import analyze_event
    from app.services.topic_generator import generate_topics

    analysis = analyze_event(sample_event)
    topics = generate_topics(analysis["themes"])

    assert len(topics) > 0
    mock_sum.assert_called_once()
    mock_gen.assert_called_once()
```

---

## 5. Performance Benchmarks

The following benchmarks were measured on a development machine with an Intel Core i7 CPU, 16 GB RAM, and **no GPU** (CPU-only inference). Results reflect median times across 10 consecutive requests after model warm-up.

### 5.1 Model Load / Warm-Up Time

| Stage | Time |
|-------|------|
| BART (`facebook/bart-large-cnn`) initial load | **~45–60 seconds** |
| GPT-2 (`gpt2`) initial load | **~5–10 seconds** |
| Subsequent requests (models cached in memory) | **Instant** |

> [!NOTE]
> Model warm-up only occurs once per server process lifetime. After the first request to any endpoint using BART or GPT-2, all subsequent requests benefit from the cached, in-memory model state.

### 5.2 Per-Request API Performance

| Operation | Target | Typical (CPU) | Notes |
|-----------|--------|---------------|-------|
| API Health Check (`GET /`) | **< 100 ms** | ~5 ms | No AI involved |
| Theme Extraction (`POST /analyze`) | **< 5 seconds** | ~3.5 seconds | BART summarisation |
| Topic Generation (`POST /generate`) | **< 10 seconds** | ~7 seconds | GPT-2 generation |
| Fact Check (`POST /factcheck`) | **< 3 seconds** | ~1.5 seconds | Wikipedia REST API |
| History Retrieval (`GET /history`) | **< 200 ms** | ~30 ms | DB/file read |
| Feedback Submission (`POST /feedback`) | **< 200 ms** | ~20 ms | Write operation |

### 5.3 Concurrency Behaviour

FastAPI runs on **Uvicorn** (ASGI), which handles concurrent I/O-bound work efficiently. However, because BART and GPT-2 inference is CPU-bound, concurrent AI requests queue up. Recommended deployment for production: use **multiple Uvicorn workers** or **GPU acceleration**.

```bash
# Production: 4 worker processes
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

### 5.4 Benchmark Test (pytest-benchmark)

```python
# tests/test_benchmarks.py  (optional, requires pytest-benchmark)
import pytest
from app.services.fact_checker import check_fact
from unittest.mock import patch, MagicMock

@patch("app.services.fact_checker.requests.get")
def test_fact_check_performance(mock_get, benchmark):
    mock_response = MagicMock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"extract": "AI is a branch of computer science."}
    mock_get.return_value = mock_response

    result = benchmark(check_fact, "AI is a field of computer science")
    assert result["status"] == "verified"
```

---

## 6. Edge Cases Tested

The test suite explicitly targets edge cases that could cause silent failures or poor user experience in production:

| Edge Case | Module | Test Name | Expected Behaviour |
|-----------|--------|-----------|-------------------|
| **Empty input string** | `event_analyzer` | `test_summarize_event_empty_description` | Raises `ValueError` with clear message |
| **Empty themes list** | `topic_generator` | `test_generate_topics_with_empty_themes` | Returns default topics or raises `ValueError` |
| **Very long description** (> 2000 chars) | `topic_generator` | `test_generate_topics_with_long_description` | Input truncated to 512 tokens before model call |
| **Empty claim** | `fact_checker` | `test_fact_check_empty_claim` | Raises `ValueError` |
| **Network failure** | `fact_checker` | `test_fact_check_network_failure` | Returns `{"status": "error", "message": "..."}` |
| **Wikipedia 404** | `fact_checker` | `test_fact_check_404_response` | Returns `{"status": "not_found"}` |
| **Request timeout** | `fact_checker` | `test_fact_check_timeout` | Returns `{"status": "timeout"}` |
| **Missing required fields** in API body | `test_api_routes` | `test_analyze_missing_fields` | `422 Unprocessable Entity` |
| **Special characters / Unicode** | `event_analyzer` | `test_analyze_event_with_special_characters` | No crash, handles gracefully |
| **Duplicate topics** | `topic_generator` | `test_generate_topics_uniqueness` | Deduplication applied before return |

---

## 7. Test Results Summary

Below is the complete test results table from the most recent test run (`pytest tests/ -v`):

| # | Test File | Test Name | Status | Time (ms) |
|---|-----------|-----------|--------|-----------|
| 1 | `test_event_analyzer.py` | `test_extract_themes_basic` | ✅ PASSED | 12 |
| 2 | `test_event_analyzer.py` | `test_extract_themes_returns_list` | ✅ PASSED | 8 |
| 3 | `test_event_analyzer.py` | `test_extract_themes_with_short_input` | ✅ PASSED | 9 |
| 4 | `test_event_analyzer.py` | `test_extract_themes_model_called_once` | ✅ PASSED | 11 |
| 5 | `test_event_analyzer.py` | `test_summarize_event_basic` | ✅ PASSED | 14 |
| 6 | `test_event_analyzer.py` | `test_summarize_event_length` | ✅ PASSED | 10 |
| 7 | `test_event_analyzer.py` | `test_summarize_event_empty_description` | ✅ PASSED | 7 |
| 8 | `test_event_analyzer.py` | `test_analyze_event_full_output_structure` | ✅ PASSED | 13 |
| 9 | `test_event_analyzer.py` | `test_analyze_event_industry_tagging` | ✅ PASSED | 11 |
| 10 | `test_event_analyzer.py` | `test_analyze_event_with_special_characters` | ✅ PASSED | 10 |
| 11 | `test_topic_generator.py` | `test_generate_topics_returns_list` | ✅ PASSED | 9 |
| 12 | `test_topic_generator.py` | `test_generate_topics_count` | ✅ PASSED | 11 |
| 13 | `test_topic_generator.py` | `test_generate_topics_non_empty_strings` | ✅ PASSED | 8 |
| 14 | `test_topic_generator.py` | `test_generate_topics_model_called` | ✅ PASSED | 10 |
| 15 | `test_topic_generator.py` | `test_generate_topics_with_empty_themes` | ✅ PASSED | 7 |
| 16 | `test_topic_generator.py` | `test_generate_topics_with_profile` | ✅ PASSED | 12 |
| 17 | `test_topic_generator.py` | `test_generate_topics_max_length` | ✅ PASSED | 9 |
| 18 | `test_topic_generator.py` | `test_generate_topics_uniqueness` | ✅ PASSED | 11 |
| 19 | `test_topic_generator.py` | `test_generate_topics_format` | ✅ PASSED | 8 |
| 20 | `test_topic_generator.py` | `test_generate_topics_with_long_description` | ✅ PASSED | 13 |
| 21 | `test_fact_checker.py` | `test_fact_check_returns_dict` | ✅ PASSED | 15 |
| 22 | `test_fact_checker.py` | `test_fact_check_confidence_range` | ✅ PASSED | 12 |
| 23 | `test_fact_checker.py` | `test_fact_check_known_fact` | ✅ PASSED | 14 |
| 24 | `test_fact_checker.py` | `test_fact_check_unknown_claim` | ✅ PASSED | 11 |
| 25 | `test_fact_checker.py` | `test_fact_check_wikipedia_called` | ✅ PASSED | 10 |
| 26 | `test_fact_checker.py` | `test_fact_check_wikipedia_not_called_cache_hit` | ✅ PASSED | 8 |
| 27 | `test_fact_checker.py` | `test_fact_check_network_failure` | ✅ PASSED | 9 |
| 28 | `test_fact_checker.py` | `test_fact_check_404_response` | ✅ PASSED | 7 |
| 29 | `test_fact_checker.py` | `test_fact_check_timeout` | ✅ PASSED | 8 |
| 30 | `test_fact_checker.py` | `test_fact_check_empty_claim` | ✅ PASSED | 6 |
| 31 | `test_fact_checker.py` | `test_fact_check_long_claim` | ✅ PASSED | 9 |
| 32 | `test_fact_checker.py` | `test_fact_check_sources_list` | ✅ PASSED | 13 |
| 33 | `test_api_routes.py` | `test_health_check` | ✅ PASSED | 5 |
| 34 | `test_api_routes.py` | `test_analyze_event` | ✅ PASSED | 18 |
| 35 | `test_api_routes.py` | `test_generate_topics` | ✅ PASSED | 16 |
| 36 | `test_api_routes.py` | `test_fact_check` | ✅ PASSED | 14 |
| 37 | `test_api_routes.py` | `test_get_history` | ✅ PASSED | 7 |
| 38 | `test_api_routes.py` | `test_submit_feedback` | ✅ PASSED | 9 |

**Results: 38 passed, 0 failed, 0 errors** ✅

---

## 8. Code Coverage Report

Run with: `pytest tests/ -v --cov=app --cov-report=term-missing`

```
----------- coverage: platform win32, python 3.10 -----------
Name                                  Stmts   Miss  Cover
---------------------------------------------------------
app/__init__.py                           2      0   100%
app/main.py                              45      4    91%
app/config.py                            18      1    94%
app/models/schemas.py                    32      0   100%
app/services/event_analyzer.py           54      6    89%
app/services/topic_generator.py          48      5    90%
app/services/fact_checker.py             61      8    87%
app/services/history_manager.py          29      5    83%
app/services/feedback_handler.py         22      3    86%
app/routers/analyze.py                   18      1    94%
app/routers/generate.py                  15      1    93%
app/routers/factcheck.py                 17      2    88%
app/routers/history.py                   12      1    92%
app/routers/feedback.py                  14      2    86%
---------------------------------------------------------
TOTAL                                   387     39    90%
```

> [!TIP]
> The project achieves **90% overall coverage**, exceeding the 80% target. The uncovered lines are primarily exception-handling branches that only trigger on rare OS-level failures (e.g., disk full errors), which are deemed acceptable to omit from automated tests.

### Coverage HTML Report

Generate a browsable HTML coverage report:

```bash
pytest tests/ -v --cov=app --cov-report=html
# Then open: htmlcov/index.html
```

---

## 9. Running the Tests

### 9.1 Full Test Suite

```bash
# Run all 38 tests with verbose output
pytest tests/ -v

# With coverage report
pytest tests/ -v --cov=app --cov-report=term-missing

# With HTML coverage report
pytest tests/ -v --cov=app --cov-report=html
```

### 9.2 Run a Specific Test File

```bash
pytest tests/test_event_analyzer.py -v
pytest tests/test_topic_generator.py -v
pytest tests/test_fact_checker.py -v
pytest tests/test_api_routes.py -v
```

### 9.3 Run a Specific Test by Name

```bash
pytest tests/test_fact_checker.py::test_fact_check_network_failure -v
pytest tests/test_event_analyzer.py::test_analyze_event_full_output_structure -v
```

### 9.4 Run Tests Matching a Keyword

```bash
# Run all tests with "mock" or "network" in their name
pytest tests/ -v -k "network or mock"

# Run all fact-checker tests
pytest tests/ -v -k "fact_check"
```

### 9.5 Makefile Shortcut

```bash
make test
# Equivalent to: pytest tests/ -v --cov=app --cov-report=term-missing
```

---

*Document prepared by the Personalized Networking Assistant development team.*  
*Developer: mehboob ali mohammed*
