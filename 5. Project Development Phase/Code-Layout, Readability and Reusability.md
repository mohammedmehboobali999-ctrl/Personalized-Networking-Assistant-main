# Code Layout, Readability and Reusability

**Project:** Personalized Networking Assistant  
**Developer:** Mehboob Ali Mohammed 
**Team Members:**  Mehboob Ali Mohammed 
**Document Version:** 1.0  
**Date:** July 2026

---

## Table of Contents

1. [Directory Structure](#1-directory-structure)
2. [Module Organization & Single Responsibility](#2-module-organization--single-responsibility)
3. [Naming Conventions](#3-naming-conventions)
4. [Code Style & PEP 8 Compliance](#4-code-style--pep-8-compliance)
5. [Documentation Standards](#5-documentation-standards)
6. [Type Hints Usage](#6-type-hints-usage)
7. [Import Organization](#7-import-organization)
8. [Reusability Patterns](#8-reusability-patterns)
9. [Service Composition & Wrapping](#9-service-composition--wrapping)
10. [Example Code Showcasing Best Practices](#10-example-code-showcasing-best-practices)

---

## 1. Directory Structure

The project follows a clean, layered architecture separating the API layer, business services, data persistence, UI, configuration, and tests. The complete file tree is shown below:

```
networking-assistant/
│
├── README.md                          # Project overview, setup instructions, usage
├── requirements.txt                   # All Python dependencies pinned to exact versions
├── Dockerfile                         # Container image definition for production
├── docker-compose.yml                 # Orchestrates FastAPI + Streamlit containers
├── .env                               # Environment variables (not committed to VCS)
├── .env.example                       # Template showing required env variables
├── .gitignore                         # Excludes __pycache__, .env, data/, models/
├── pytest.ini                         # Pytest configuration, coverage settings
│
├── app/                               # Core application package
│   ├── __init__.py                    # Package marker; exports version constant
│   │
│   ├── config.py                      # Centralized Settings via pydantic-settings
│   │
│   ├── main.py                        # FastAPI application factory & router inclusion
│   │
│   ├── routers/                       # HTTP layer — thin route handlers only
│   │   ├── __init__.py
│   │   ├── events.py                  # POST /analyze-event
│   │   ├── topics.py                  # POST /generate-topics
│   │   ├── fact_check.py              # POST /fact-check
│   │   ├── history.py                 # GET  /history, DELETE /history
│   │   └── feedback.py                # POST /feedback, GET /feedback
│   │
│   ├── services/                      # Business logic layer — pure Python, no HTTP
│   │   ├── __init__.py
│   │   ├── nlp_service.py             # Lazy-loaded BART + GPT-2 model singletons
│   │   ├── event_analyzer.py          # Theme extraction using BART (wraps nlp_service)
│   │   ├── topic_generator.py         # Starter generation using GPT-2 (wraps nlp_service)
│   │   ├── fact_checker.py            # Wikipedia REST API fact-checking
│   │   ├── history_logger.py          # JSON-based conversation persistence
│   │   └── feedback_logger.py         # JSON-based feedback persistence
│   │
│   ├── models/                        # Pydantic request/response schemas
│   │   ├── __init__.py
│   │   ├── event_models.py            # EventRequest, EventResponse
│   │   ├── topic_models.py            # TopicRequest, TopicResponse
│   │   ├── fact_check_models.py       # FactCheckRequest, FactCheckResponse
│   │   ├── history_models.py          # ConversationEntry, HistoryResponse
│   │   └── feedback_models.py         # FeedbackEntry, FeedbackResponse
│   │
│   └── utils/                         # Shared utilities, no business logic
│       ├── __init__.py
│       ├── text_utils.py              # Text cleaning, truncation helpers
│       └── time_utils.py              # Timezone-aware UTC timestamp helpers
│
├── streamlit_app/                     # Streamlit frontend package
│   ├── __init__.py
│   ├── app.py                         # Entry point: page routing via session_state
│   ├── pages/
│   │   ├── home.py                    # Home page: event input + starter display
│   │   ├── dashboard.py               # Analytics charts and summary cards
│   │   └── history.py                 # History viewer with search and expand
│   └── components/
│       ├── sidebar.py                 # Shared sidebar with navigation
│       ├── starter_card.py            # Reusable starter display component
│       └── feedback_widget.py         # Like/dislike + star rating widget
│
├── data/                              # Runtime data (git-ignored)
│   ├── history/
│   │   └── conversations.json         # Persisted conversation sessions
│   └── feedback/
│       └── feedback_log.json          # Persisted feedback entries
│
└── tests/                             # Test suite (38 tests)
    ├── __init__.py
    ├── conftest.py                    # Shared fixtures: test client, mock models
    ├── unit/
    │   ├── test_event_analyzer.py     # 8 unit tests for theme extraction
    │   ├── test_topic_generator.py    # 7 unit tests for topic generation
    │   ├── test_fact_checker.py       # 6 unit tests for Wikipedia API
    │   ├── test_history_logger.py     # 5 unit tests for history CRUD
    │   └── test_feedback_logger.py    # 5 unit tests for feedback CRUD
    └── integration/
        ├── test_events_router.py      # 3 integration tests (HTTP layer)
        ├── test_topics_router.py      # 2 integration tests
        └── test_fact_check_router.py  # 2 integration tests
```

> **Principle:** Every file exists for exactly one reason. Adding a new feature means adding a new service file, a new model file, and a new router file — existing files remain untouched.

---

## 2. Module Organization & Single Responsibility

Each module is assigned a single, well-defined responsibility. This adheres to the **Single Responsibility Principle (SRP)** of SOLID design.

| Module | Layer | Sole Responsibility |
|--------|-------|---------------------|
| `app/config.py` | Config | Load and validate all environment variables; expose typed settings |
| `app/main.py` | API Entry | Create `FastAPI` instance, include routers, add middleware |
| `app/routers/events.py` | HTTP | Parse HTTP request → call service → return HTTP response |
| `app/services/nlp_service.py` | Service | Own the lifecycle of BART and GPT-2 model objects |
| `app/services/event_analyzer.py` | Service | Implement theme extraction algorithm using the NLP service |
| `app/services/topic_generator.py` | Service | Implement starter generation algorithm using the NLP service |
| `app/services/fact_checker.py` | Service | Query Wikipedia API and interpret confidence |
| `app/services/history_logger.py` | Service | Read/write conversation history to disk atomically |
| `app/services/feedback_logger.py` | Service | Read/write feedback entries to disk atomically |
| `app/models/event_models.py` | Schema | Define and validate the shape of event-related data |
| `app/utils/text_utils.py` | Utility | Provide pure functions for string manipulation |
| `app/utils/time_utils.py` | Utility | Provide timezone-aware datetime helpers |
| `streamlit_app/pages/home.py` | UI | Render and handle the Home page layout and interactions |
| `streamlit_app/components/feedback_widget.py` | UI Component | Render the reusable feedback UI element |

### What Each Service Does NOT Do

- **Routers** do not contain business logic or direct model inference calls.
- **Services** do not import FastAPI, Starlette, or Streamlit.
- **Models (Pydantic)** do not contain logic beyond field validation.
- **Utils** do not import from `services` or `routers`.

---

## 3. Naming Conventions

### 3.1 File Names

All file names use **snake_case** (lowercase words separated by underscores). This matches Python's standard module naming convention.

```
✅ event_analyzer.py
✅ history_logger.py
✅ fact_check_models.py
✅ text_utils.py

❌ EventAnalyzer.py       # PascalCase — reserved for class names
❌ historyLogger.py       # camelCase — JavaScript convention
❌ fact-check-models.py   # Hyphens are invalid in Python module names
```

### 3.2 Class Names

All classes use **PascalCase**:

```python
# Pydantic models
class EventRequest(BaseModel): ...
class EventResponse(BaseModel): ...
class ConversationEntry(BaseModel): ...
class FeedbackEntry(BaseModel): ...

# Service classes
class NLPService: ...
class EventAnalyzer: ...
class TopicGenerator: ...
class FactChecker: ...
class HistoryLogger: ...
class FeedbackLogger: ...

# Config
class Settings(BaseSettings): ...
```

### 3.3 Function Names

All functions and methods use **snake_case** verbs that describe the action:

```python
# Services
def extract_event_themes(event_description: str) -> list[str]: ...
def generate_topics(themes: list[str], count: int) -> list[str]: ...
def fact_check(topic: str) -> dict: ...
def log_conversation(entry: ConversationEntry) -> None: ...
def load_history(limit: int, offset: int) -> list[ConversationEntry]: ...
def log_feedback(entry: FeedbackEntry) -> None: ...
def load_feedback(limit: int) -> list[FeedbackEntry]: ...

# Utils
def clean_text(text: str) -> str: ...
def truncate_text(text: str, max_length: int) -> str: ...
def get_utc_now() -> datetime: ...
def format_timestamp(dt: datetime) -> str: ...

# NLP Service (internal)
def _load_bart_model() -> pipeline: ...       # leading underscore = private
def _load_gpt2_model() -> pipeline: ...       # leading underscore = private
```

### 3.4 Variable Names

| Category | Convention | Example |
|----------|-----------|---------|
| Local variables | snake_case | `event_description`, `theme_list`, `top_k` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_THEMES`, `DEFAULT_STARTER_COUNT`, `DATA_DIR` |
| Private attributes | Leading underscore | `self._bart_pipeline`, `self._gpt2_pipeline` |
| Boolean variables | `is_`, `has_`, `can_` prefix | `is_loaded`, `has_themes`, `can_generate` |
| Collections | Plural nouns | `themes`, `starters`, `entries`, `results` |

```python
# Constants in config.py
MAX_THEMES: int = 3
DEFAULT_STARTER_COUNT: int = 3
MAX_STARTER_COUNT: int = 5
DATA_DIR: Path = Path("data")
HISTORY_FILE: Path = DATA_DIR / "history" / "conversations.json"
FEEDBACK_FILE: Path = DATA_DIR / "feedback" / "feedback_log.json"

# Boolean variable naming
is_loaded: bool = False
has_wikipedia_result: bool = result is not None
```

### 3.5 Endpoint & Route Names

REST endpoints use **kebab-case** resource paths and HTTP verbs:

```
POST   /analyze-event
POST   /generate-topics
POST   /fact-check
GET    /history
DELETE /history
POST   /feedback
GET    /feedback
```

---

## 4. Code Style & PEP 8 Compliance

### 4.1 Indentation

The project enforces **4-space indentation** throughout. Tabs are disallowed. This is configured in `pytest.ini` and enforced by the `ruff` linter.

```python
# ✅ Correct — 4-space indent
def extract_event_themes(event_description: str) -> list[str]:
    cleaned = clean_text(event_description)
    results = self._bart_pipeline(
        cleaned,
        candidate_labels=CANDIDATE_THEMES,
        multi_label=True,
    )
    return [
        label
        for label, score in zip(results["labels"], results["scores"])
        if score >= THEME_CONFIDENCE_THRESHOLD
    ][:MAX_THEMES]

# ❌ Wrong — 2-space indent
def extract_event_themes(event_description: str) -> list[str]:
  cleaned = clean_text(event_description)
  ...
```

### 4.2 Line Length

Maximum **88 characters per line** (Black formatter default). Long expressions are broken using Python's implicit line continuation inside brackets:

```python
# ✅ Long function call broken across lines
results = self._bart_pipeline(
    cleaned_description,
    candidate_labels=CANDIDATE_THEMES,
    multi_label=True,
    hypothesis_template="This event is about {}.",
)

# ✅ Long list/dict broken across lines
CANDIDATE_THEMES = [
    "technology",
    "entrepreneurship",
    "healthcare",
    "finance",
    "artificial intelligence",
    "education",
    "sustainability",
    "leadership",
    "marketing",
    "data science",
]

# ✅ Long condition broken with parentheses
if (
    score >= THEME_CONFIDENCE_THRESHOLD
    and label in CANDIDATE_THEMES
    and len(selected_themes) < MAX_THEMES
):
    selected_themes.append(label)
```

### 4.3 Blank Lines

- **Two blank lines** between top-level definitions (functions, classes).
- **One blank line** between methods inside a class.
- **One blank line** to separate logical sections inside a function.

```python
class EventAnalyzer:
    """Extracts themes from networking event descriptions using BART."""

    def __init__(self, nlp_service: NLPService) -> None:
        self._nlp = nlp_service

    def extract_event_themes(self, event_description: str) -> list[str]:
        # Preprocessing
        cleaned = clean_text(event_description)
        truncated = truncate_text(cleaned, max_length=512)

        # Model inference
        results = self._nlp.bart_pipeline(
            truncated,
            candidate_labels=CANDIDATE_THEMES,
            multi_label=True,
        )

        # Post-processing
        return self._filter_top_themes(results)

    def _filter_top_themes(self, results: dict) -> list[str]:
        pairs = zip(results["labels"], results["scores"])
        return [
            label
            for label, score in pairs
            if score >= THEME_CONFIDENCE_THRESHOLD
        ][:MAX_THEMES]
```

### 4.4 String Formatting

**f-strings** are used exclusively for string interpolation (Python 3.8+). No `%` formatting or `.format()` calls.

```python
# ✅ f-string
prompt = f"At a {theme} networking event, a great conversation starter is:"
logger.info(f"Extracted {len(themes)} themes from event description.")
raise ValueError(f"Expected 1-{MAX_STARTER_COUNT} starters, got {count}.")

# ❌ Old-style (forbidden)
prompt = "At a %s networking event, ..." % theme
prompt = "At a {} networking event, ...".format(theme)
```

### 4.5 Trailing Commas

Trailing commas are used on the last item of multi-line function arguments and collections. This produces cleaner diffs when items are added or removed:

```python
# ✅ Trailing comma — adding a new theme won't touch existing lines
CANDIDATE_THEMES = [
    "technology",
    "entrepreneurship",
    "healthcare",      # ← trailing comma
]
```

---

## 5. Documentation Standards

### 5.1 Module-Level Docstrings

Every Python file begins with a module-level docstring explaining its purpose, what it provides, and any important notes:

```python
"""
app/services/event_analyzer.py
-------------------------------
Provides the EventAnalyzer service, which uses a BART zero-shot classification
pipeline to extract the most relevant themes from a networking event description.

The analyzer is designed to be instantiated once (via dependency injection in
the router) and reused across requests. Model loading is handled lazily by
NLPService to avoid startup latency.

Typical usage::

    analyzer = EventAnalyzer(nlp_service=get_nlp_service())
    themes = analyzer.extract_event_themes("A fintech demo day for investors.")
    # themes → ["finance", "entrepreneurship", "technology"]
"""
```

### 5.2 Google-Style Docstrings

All public functions and methods carry **Google-style docstrings** with `Args`, `Returns`, and `Raises` sections:

```python
def extract_event_themes(self, event_description: str) -> list[str]:
    """Extract the top networking themes from an event description.

    Uses BART zero-shot classification to score a fixed set of candidate
    theme labels against the provided description. Returns up to MAX_THEMES
    labels whose confidence score exceeds THEME_CONFIDENCE_THRESHOLD.

    Args:
        event_description: A plain-text description of the networking event.
            Should be between 10 and 512 characters. Shorter inputs may
            produce low-confidence results.

    Returns:
        A list of theme strings, ordered by confidence score (highest first).
        Contains between 1 and MAX_THEMES (3) items. Returns ["general"] if
        no theme exceeds the confidence threshold.

    Raises:
        ValueError: If event_description is empty or contains only whitespace.
        RuntimeError: If the BART model fails to load or inference fails.

    Example::

        >>> analyzer.extract_event_themes("AI startup pitch night")
        ["artificial intelligence", "entrepreneurship", "technology"]
    """
```

```python
def log_conversation(self, entry: ConversationEntry) -> None:
    """Append a conversation session to the persistent JSON history file.

    Reads the existing history from disk, appends the new entry, and writes
    the updated list atomically. Creates the history file if it does not exist.

    Args:
        entry: A ConversationEntry Pydantic model containing the session ID,
            timestamp, event description, extracted themes, and generated
            conversation starters.

    Returns:
        None

    Raises:
        IOError: If the history file cannot be read or written due to
            filesystem permissions or disk space issues.
    """
```

### 5.3 Inline Comments

Inline comments explain **why**, not **what**. They are used sparingly for non-obvious logic:

```python
# GPT-2 is an auto-regressive model; it can repeat the prompt in the output.
# We strip everything up to and including the prompt prefix before returning.
raw_output = self._nlp.gpt2_pipeline(prompt, max_new_tokens=60)[0]["generated_text"]
starter = raw_output[len(prompt):].strip()

# Wikipedia API raises DisambiguationError when the topic matches multiple articles.
# We catch it and return the first suggestion rather than failing the request.
try:
    page = wiki.page(topic)
except wikipedia.DisambiguationError as e:
    page = wiki.page(e.options[0])
```

### 5.4 TODO Comments

TODOs use the standard format `# TODO(assignee): description`:

```python
# TODO(sumiya): Add caching layer (Redis) to avoid redundant Wikipedia calls.
# TODO(tejesh): Replace file-based storage with SQLite for concurrent writes.
```

---

## 6. Type Hints Usage

Every public function signature carries full type annotations. The project targets **Python 3.10+** and uses the built-in generic types (`list[str]`, `dict[str, Any]`) instead of `typing.List`, `typing.Dict`.

### 6.1 Function Signatures

```python
# services/event_analyzer.py
def extract_event_themes(self, event_description: str) -> list[str]: ...

# services/topic_generator.py
def generate_topics(
    self,
    themes: list[str],
    count: int = DEFAULT_STARTER_COUNT,
) -> list[str]: ...

# services/fact_checker.py
def fact_check(self, topic: str) -> dict[str, Any]: ...

# services/history_logger.py
def log_conversation(self, entry: ConversationEntry) -> None: ...
def load_history(self, limit: int = 20, offset: int = 0) -> list[ConversationEntry]: ...

# services/feedback_logger.py
def log_feedback(self, entry: FeedbackEntry) -> None: ...
def load_feedback(self, limit: int = 10) -> list[FeedbackEntry]: ...

# utils/text_utils.py
def clean_text(text: str) -> str: ...
def truncate_text(text: str, max_length: int = 512) -> str: ...

# utils/time_utils.py
def get_utc_now() -> datetime: ...
def format_timestamp(dt: datetime) -> str: ...
```

### 6.2 Pydantic Models (Runtime Type Enforcement)

```python
from pydantic import BaseModel, Field
from datetime import datetime
from typing import Optional

class ConversationEntry(BaseModel):
    session_id: str = Field(..., description="Unique UUID for this session")
    timestamp: datetime = Field(..., description="UTC datetime of the session")
    event_description: str = Field(..., min_length=10, max_length=1000)
    themes: list[str] = Field(..., min_length=1, max_length=3)
    starters: list[str] = Field(..., min_length=1, max_length=5)

class FeedbackEntry(BaseModel):
    session_id: str
    starter_index: int = Field(..., ge=0, le=4)
    liked: Optional[bool] = None
    rating: Optional[int] = Field(None, ge=1, le=5)
    comment: Optional[str] = Field(None, max_length=500)
    timestamp: datetime
```

### 6.3 Optional and Union Types

```python
from typing import Optional, Union

# Optional return when data may not exist
def get_wikipedia_summary(topic: str) -> Optional[str]: ...

# Union for flexible input
def parse_theme_input(themes: Union[str, list[str]]) -> list[str]: ...
```

### 6.4 Type Aliases

```python
from typing import TypeAlias

ThemeList: TypeAlias = list[str]
StarterList: TypeAlias = list[str]
JsonDict: TypeAlias = dict[str, Any]
```

---

## 7. Import Organization

Imports in every file follow the **three-block structure** defined by PEP 8 and enforced by `isort`:

```
Block 1: Standard library imports (alphabetical)
[blank line]
Block 2: Third-party library imports (alphabetical)
[blank line]
Block 3: Local application imports (alphabetical)
```

### 7.1 Example — `event_analyzer.py`

```python
# ── Standard Library ──────────────────────────────────────────────────────────
import logging
from typing import Any

# ── Third-Party ───────────────────────────────────────────────────────────────
from transformers import pipeline

# ── Local Application ─────────────────────────────────────────────────────────
from app.config import settings
from app.services.nlp_service import NLPService
from app.utils.text_utils import clean_text, truncate_text

logger = logging.getLogger(__name__)
```

### 7.2 Example — `main.py`

```python
# ── Standard Library ──────────────────────────────────────────────────────────
import logging

# ── Third-Party ───────────────────────────────────────────────────────────────
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

# ── Local Application ─────────────────────────────────────────────────────────
from app.config import settings
from app.routers import events, fact_check, feedback, history, topics
```

### 7.3 Example — `home.py` (Streamlit)

```python
# ── Standard Library ──────────────────────────────────────────────────────────
import uuid
from datetime import datetime

# ── Third-Party ───────────────────────────────────────────────────────────────
import httpx
import streamlit as st

# ── Local Application ─────────────────────────────────────────────────────────
from streamlit_app.components.feedback_widget import render_feedback_widget
from streamlit_app.components.starter_card import render_starter_card
```

---

## 8. Reusability Patterns

### 8.1 Singleton Pattern — `NLPService`

The two heavy ML models (BART and GPT-2) are loaded exactly **once** per process, regardless of how many requests arrive. `NLPService` implements the Singleton pattern using a class-level instance variable:

```python
class NLPService:
    """Manages BART and GPT-2 pipelines as process-wide singletons.

    Models are loaded lazily on first access to avoid blocking application
    startup. Subsequent accesses return the already-loaded pipeline objects.
    """

    _instance: Optional["NLPService"] = None
    _bart: Optional[pipeline] = None
    _gpt2: Optional[pipeline] = None

    def __new__(cls) -> "NLPService":
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    @property
    def bart_pipeline(self) -> pipeline:
        """Return the BART zero-shot classification pipeline, loading it if needed."""
        if self._bart is None:
            logger.info("Loading BART model — this may take 20-30 seconds...")
            self._bart = pipeline(
                "zero-shot-classification",
                model="facebook/bart-large-mnli",
            )
            logger.info("BART model loaded successfully.")
        return self._bart

    @property
    def gpt2_pipeline(self) -> pipeline:
        """Return the GPT-2 text generation pipeline, loading it if needed."""
        if self._gpt2 is None:
            logger.info("Loading GPT-2 model...")
            self._gpt2 = pipeline(
                "text-generation",
                model="gpt2",
                pad_token_id=50256,
            )
            logger.info("GPT-2 model loaded successfully.")
        return self._gpt2


def get_nlp_service() -> NLPService:
    """FastAPI dependency that returns the singleton NLPService."""
    return NLPService()
```

### 8.2 Lazy Loading — Property-Based Model Access

Instead of loading models in `__init__`, the project uses `@property` accessors that load on **first use**. This means:
- The app starts instantly, even on machines where model download takes minutes.
- Unit tests that mock `nlp_service` never trigger a model download.
- Memory is only allocated when the relevant endpoint is actually called.

### 8.3 Wrapper Pattern — Service Composition

Higher-level services wrap `NLPService`, providing a clean interface without exposing pipeline internals. Routers only ever call high-level service methods:

```
Router (HTTP layer)
    ↓ calls
EventAnalyzer.extract_event_themes()      ← domain logic
    ↓ delegates to
NLPService.bart_pipeline(...)             ← ML infrastructure
    ↓ delegates to
transformers.pipeline(...)                ← third-party library
```

### 8.4 Dependency Injection — FastAPI `Depends`

Services are injected into routers via FastAPI's `Depends` mechanism. This makes routers unit-testable by swapping in mock services without modifying any production code:

```python
# routers/events.py
from fastapi import APIRouter, Depends
from app.services.event_analyzer import EventAnalyzer
from app.services.nlp_service import get_nlp_service

router = APIRouter(prefix="/analyze-event", tags=["Events"])

@router.post("/", response_model=EventResponse)
async def analyze_event(
    request: EventRequest,
    nlp_service: NLPService = Depends(get_nlp_service),
) -> EventResponse:
    analyzer = EventAnalyzer(nlp_service=nlp_service)
    themes = analyzer.extract_event_themes(request.event_description)
    return EventResponse(themes=themes)
```

```python
# tests/unit/test_events_router.py — swapping the dependency in tests
def test_analyze_event_returns_themes(client, mock_nlp_service):
    app.dependency_overrides[get_nlp_service] = lambda: mock_nlp_service
    response = client.post("/analyze-event/", json={"event_description": "AI summit"})
    assert response.status_code == 200
    assert "themes" in response.json()
```

### 8.5 Reusable Streamlit Components

UI elements that appear on multiple pages (feedback widget, starter card) are extracted into `streamlit_app/components/` and imported wherever needed:

```python
# components/starter_card.py
def render_starter_card(
    index: int,
    starter: str,
    session_id: str,
    *,
    show_feedback: bool = True,
) -> None:
    """Render a single conversation starter with optional feedback controls."""
    with st.container():
        st.markdown(f"**{index + 1}.** {starter}")
        if show_feedback:
            render_feedback_widget(
                session_id=session_id,
                starter_index=index,
                starter_text=starter,
            )
```

---

## 9. Service Composition & Wrapping

The following diagram illustrates how services compose and delegate:

```
┌─────────────────────────────────────────────────────────────────┐
│                        ROUTERS (HTTP Layer)                      │
│  events.py   topics.py   fact_check.py   history.py  feedback.py│
└────────┬──────────┬───────────┬───────────────────────────────-─┘
         │          │           │
         ▼          ▼           ▼
┌────────────┐ ┌───────────┐ ┌────────────┐ ┌──────────────────┐
│ EventAna-  │ │ TopicGene-│ │ FactChecker│ │  HistoryLogger / │
│ lyzer      │ │ rator     │ │            │ │  FeedbackLogger  │
│            │ │           │ │            │ │                  │
│ (wraps NLP)│ │(wraps NLP)│ │(wraps wiki)│ │ (wraps JSON I/O) │
└─────┬──────┘ └─────┬─────┘ └──────┬─────┘ └──────────────────┘
      │              │              │
      └──────┬────────┘             │
             ▼                      ▼
     ┌───────────────┐     ┌────────────────┐
     │  NLPService   │     │  wikipedia SDK │
     │  (Singleton)  │     │  + REST API    │
     │               │     │                │
     │  BART  GPT-2  │     │ en.wikipedia   │
     └───────────────┘     └────────────────┘
```

**Key composition rules:**
1. Routers instantiate service objects using injected dependencies.
2. `EventAnalyzer` and `TopicGenerator` both receive `NLPService` but expose different domain methods.
3. `FactChecker` wraps the `wikipedia` library and adds confidence scoring on top.
4. `HistoryLogger` and `FeedbackLogger` both wrap raw JSON I/O with atomic read-modify-write operations.

---

## 10. Example Code Showcasing Best Practices

### 10.1 Clean, Documented Service — `fact_checker.py`

```python
"""
app/services/fact_checker.py
------------------------------
Provides FactChecker, which queries the Wikipedia REST API to verify whether
a given topic or claim is supported by a Wikipedia article. Returns a
structured response containing the summary, confidence level, and article URL.
"""

# ── Standard Library ──────────────────────────────────────────────────────────
import logging
from typing import Any, Optional

# ── Third-Party ───────────────────────────────────────────────────────────────
import wikipedia

# ── Local Application ─────────────────────────────────────────────────────────
from app.config import settings
from app.utils.text_utils import truncate_text

logger = logging.getLogger(__name__)

CONFIDENCE_HIGH = "high"
CONFIDENCE_MEDIUM = "medium"
CONFIDENCE_LOW = "low"
SUMMARY_MAX_CHARS = 600


class FactChecker:
    """Checks facts against Wikipedia and returns a confidence-scored result."""

    def __init__(self) -> None:
        wikipedia.set_lang("en")
        wikipedia.set_user_agent(settings.wikipedia_user_agent)

    def fact_check(self, topic: str) -> dict[str, Any]:
        """Verify a topic against Wikipedia and return a structured result.

        Args:
            topic: The networking-related topic or claim to verify.

        Returns:
            A dict with keys:
                - ``found`` (bool): Whether a Wikipedia article was found.
                - ``summary`` (str): First paragraph of the article, truncated.
                - ``url`` (str): Full Wikipedia URL for the article.
                - ``confidence`` (str): One of "high", "medium", or "low".

        Raises:
            ValueError: If topic is empty or contains only whitespace.
        """
        if not topic or not topic.strip():
            raise ValueError("topic must be a non-empty string.")

        try:
            page = self._fetch_page(topic)
            summary = truncate_text(page.summary, SUMMARY_MAX_CHARS)
            confidence = self._score_confidence(page, topic)
            return {
                "found": True,
                "summary": summary,
                "url": page.url,
                "confidence": confidence,
            }
        except wikipedia.PageError:
            logger.warning(f"No Wikipedia page found for topic: '{topic}'")
            return {"found": False, "summary": "", "url": "", "confidence": CONFIDENCE_LOW}

    def _fetch_page(self, topic: str) -> wikipedia.WikipediaPage:
        """Fetch a Wikipedia page, handling disambiguation automatically."""
        try:
            return wikipedia.page(topic, auto_suggest=False)
        except wikipedia.DisambiguationError as e:
            logger.info(f"Disambiguation for '{topic}'; using first option: {e.options[0]}")
            return wikipedia.page(e.options[0], auto_suggest=False)

    def _score_confidence(
        self, page: wikipedia.WikipediaPage, topic: str
    ) -> str:
        """Score confidence based on title match and summary length."""
        title_matches = topic.lower() in page.title.lower()
        summary_is_long = len(page.summary) > 300

        if title_matches and summary_is_long:
            return CONFIDENCE_HIGH
        if title_matches or summary_is_long:
            return CONFIDENCE_MEDIUM
        return CONFIDENCE_LOW
```

### 10.2 Thin Router — `events.py`

```python
"""
app/routers/events.py
----------------------
HTTP route handler for event theme extraction.
Contains no business logic — delegates entirely to EventAnalyzer.
"""

# ── Standard Library ──────────────────────────────────────────────────────────
import logging

# ── Third-Party ───────────────────────────────────────────────────────────────
from fastapi import APIRouter, Depends, HTTPException, status

# ── Local Application ─────────────────────────────────────────────────────────
from app.models.event_models import EventRequest, EventResponse
from app.services.event_analyzer import EventAnalyzer
from app.services.nlp_service import NLPService, get_nlp_service

logger = logging.getLogger(__name__)
router = APIRouter(prefix="/analyze-event", tags=["Events"])


@router.post(
    "/",
    response_model=EventResponse,
    summary="Extract themes from an event description",
    status_code=status.HTTP_200_OK,
)
async def analyze_event(
    request: EventRequest,
    nlp_service: NLPService = Depends(get_nlp_service),
) -> EventResponse:
    """Extract up to 3 networking themes from the given event description.

    Args:
        request: JSON body containing ``event_description``.
        nlp_service: Injected singleton NLP service.

    Returns:
        JSON body containing ``themes`` list.

    Raises:
        422: If the request body fails Pydantic validation.
        500: If model inference fails unexpectedly.
    """
    try:
        analyzer = EventAnalyzer(nlp_service=nlp_service)
        themes = analyzer.extract_event_themes(request.event_description)
        logger.info(f"Extracted themes {themes} for session request.")
        return EventResponse(themes=themes)
    except RuntimeError as exc:
        logger.exception("Model inference failed during event analysis.")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Model inference error: {exc}",
        ) from exc
```

### 10.3 Reusable Utility — `text_utils.py`

```python
"""
app/utils/text_utils.py
------------------------
Pure functions for text preprocessing. No side effects, no imports from
other application modules. Safe to use from any layer.
"""

# ── Standard Library ──────────────────────────────────────────────────────────
import re
import unicodedata


def clean_text(text: str) -> str:
    """Normalize and clean input text for NLP processing.

    - Normalizes Unicode to NFC form.
    - Collapses multiple whitespace characters into a single space.
    - Strips leading and trailing whitespace.
    - Removes non-printable control characters.

    Args:
        text: Raw input string from the user or API.

    Returns:
        Cleaned, normalized string ready for tokenization.
    """
    normalized = unicodedata.normalize("NFC", text)
    no_control = re.sub(r"[\x00-\x1f\x7f]", " ", normalized)
    collapsed = re.sub(r"\s+", " ", no_control)
    return collapsed.strip()


def truncate_text(text: str, max_length: int = 512) -> str:
    """Truncate text to a maximum number of characters at a word boundary.

    Args:
        text: Input string to truncate.
        max_length: Maximum allowed length in characters. Defaults to 512.

    Returns:
        Truncated string. If text is already within max_length, returned as-is.
    """
    if len(text) <= max_length:
        return text
    truncated = text[:max_length].rsplit(" ", 1)[0]
    return truncated
```

---

## Summary

| Principle | Implementation |
|-----------|---------------|
| Single Responsibility | One class/module per concept; 14 distinct modules |
| Open/Closed | Add features by adding files; existing code unchanged |
| Dependency Inversion | FastAPI `Depends` injects services; routers depend on abstractions |
| DRY (Don't Repeat Yourself) | `NLPService` singleton; shared Streamlit components; `text_utils` |
| KISS (Keep It Simple) | Thin routers, simple JSON storage, no ORM overhead |
| PEP 8 | Enforced by `ruff` linter and `black` formatter in CI |
| Type Safety | Full type hints + Pydantic runtime validation |
| Documentation | Google-style docstrings on every public function |

---

*Document prepared by the Personalized Networking Assistant team, July 2026.*
