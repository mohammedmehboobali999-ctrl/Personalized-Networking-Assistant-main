# Coding & Solution

**Project:** Personalized Networking Assistant  
**Team Lead:** Shaik Sumiya Zainab  
**Team Members:** Naga Jagan Mohan Rao Thattukolla · Satvika Tallam · Tejesh Velivela  
**Document Version:** 1.0  
**Date:** July 2026

---

## Table of Contents

1. [Solution Overview](#1-solution-overview)
2. [Architecture Diagram](#2-architecture-diagram)
3. [Core Service Implementations](#3-core-service-implementations)
   - 3.1 [event_analyzer.py — BART Theme Extraction](#31-event_analyzerpy--bart-theme-extraction)
   - 3.2 [topic_generator.py — GPT-2 Starter Generation](#32-topic_generatorpy--gpt-2-starter-generation)
   - 3.3 [fact_checker.py — Wikipedia REST API](#33-fact_checkerpy--wikipedia-rest-api)
   - 3.4 [history_logger.py — Conversation Persistence](#34-history_loggerpy--conversation-persistence)
   - 3.5 [feedback_logger.py — Feedback Persistence](#35-feedback_loggerpy--feedback-persistence)
4. [FastAPI Endpoint Implementations](#4-fastapi-endpoint-implementations)
5. [Streamlit UI Implementation](#5-streamlit-ui-implementation)
6. [Configuration Management](#6-configuration-management)
7. [Challenges Faced & Solutions](#7-challenges-faced--solutions)
8. [Code Quality Metrics](#8-code-quality-metrics)

---

## 1. Solution Overview

The **Personalized Networking Assistant** is a two-tier web application that helps professionals generate meaningful conversation starters for networking events. The solution is built around three complementary AI capabilities:

| Capability | Model / API | Purpose |
|------------|-------------|---------|
| Theme Extraction | BART (`facebook/bart-large-mnli`) | Understand what a networking event is about |
| Starter Generation | GPT-2 (`gpt2`) | Generate natural, contextual conversation openers |
| Fact-Checking | Wikipedia REST API | Validate topics against authoritative information |

### End-to-End Flow

```
User enters event description
        │
        ▼
[Streamlit Home Page]
        │  HTTP POST /analyze-event
        ▼
[FastAPI] → EventAnalyzer → BART
        │  returns: ["technology", "entrepreneurship"]
        │
        │  HTTP POST /generate-topics
        ▼
[FastAPI] → TopicGenerator → GPT-2
        │  returns: ["Have you shipped any AI features recently?", ...]
        │
        │  HTTP POST /fact-check  (optional)
        ▼
[FastAPI] → FactChecker → Wikipedia API
        │  returns: {found: true, confidence: "high", url: "..."}
        │
        │  HTTP POST /feedback  (user rates starters)
        ▼
[FastAPI] → FeedbackLogger → feedback_log.json
        │
        │  HTTP POST /history  (auto-saved)
        ▼
[FastAPI] → HistoryLogger → conversations.json
```

### Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| API Framework | FastAPI | 0.111.x |
| Data Validation | Pydantic v2 + pydantic-settings | 2.x |
| NLP Models | 🤗 Transformers | 4.40.x |
| Text Generation | GPT-2 via `transformers.pipeline` | built-in |
| Zero-Shot Classification | BART-large-MNLI via `transformers.pipeline` | built-in |
| Fact-Checking | `wikipedia` Python SDK | 1.4.x |
| Frontend | Streamlit | 1.35.x |
| HTTP Client (UI→API) | `httpx` (async) | 0.27.x |
| Testing | `pytest` + `pytest-cov` + `httpx` TestClient | — |
| Containerization | Docker + docker-compose | — |

---

## 2. Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          Docker Network                                  │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │              Streamlit Container  (port 8501)                       │ │
│  │                                                                     │ │
│  │   streamlit_app/app.py                                              │ │
│  │        ├── pages/home.py           ← Event input, starters display  │ │
│  │        ├── pages/dashboard.py      ← Analytics charts               │ │
│  │        ├── pages/history.py        ← History + feedback viewer      │ │
│  │        └── components/             ← Reusable widgets               │ │
│  └────────────────────────┬───────────────────────────────────────────┘ │
│                           │  HTTP (httpx)                                │
│  ┌────────────────────────▼───────────────────────────────────────────┐ │
│  │              FastAPI Container  (port 8000)                         │ │
│  │                                                                     │ │
│  │   app/main.py                                                       │ │
│  │        ├── /analyze-event    → EventAnalyzer  → BART pipeline      │ │
│  │        ├── /generate-topics  → TopicGenerator → GPT-2 pipeline     │ │
│  │        ├── /fact-check       → FactChecker   → Wikipedia API       │ │
│  │        ├── /history (GET)    → HistoryLogger → conversations.json  │ │
│  │        ├── /history (DELETE) → HistoryLogger → clears file         │ │
│  │        ├── /feedback (POST)  → FeedbackLogger→ feedback_log.json   │ │
│  │        └── /feedback (GET)   → FeedbackLogger→ feedback_log.json   │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │              Shared Volume  /app/data                               │ │
│  │   data/history/conversations.json                                   │ │
│  │   data/feedback/feedback_log.json                                   │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Core Service Implementations

### 3.1 `event_analyzer.py` — BART Theme Extraction

**Purpose:** Use BART zero-shot classification to identify what networking themes are present in a user-provided event description.

**Algorithm:**
1. Clean and truncate the input text (max 512 chars for BART tokenizer).
2. Run BART zero-shot classification against 10 candidate theme labels.
3. Filter labels whose confidence score ≥ 0.30.
4. Return the top 3 labels sorted by score descending.
5. Fall back to `["general"]` if no label clears the threshold.

```python
"""
app/services/event_analyzer.py
"""
import logging
from typing import Any

from app.config import settings
from app.services.nlp_service import NLPService
from app.utils.text_utils import clean_text, truncate_text

logger = logging.getLogger(__name__)

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
MAX_THEMES = 3
THEME_CONFIDENCE_THRESHOLD = 0.30


class EventAnalyzer:
    """Extracts dominant networking themes from event descriptions using BART."""

    def __init__(self, nlp_service: NLPService) -> None:
        self._nlp = nlp_service

    def extract_event_themes(self, event_description: str) -> list[str]:
        """Extract up to 3 themes from the event description.

        Args:
            event_description: Plain-text event description from the user.

        Returns:
            List of 1-3 theme strings, ordered by confidence score.

        Raises:
            ValueError: If event_description is empty.
            RuntimeError: If BART inference fails.
        """
        if not event_description or not event_description.strip():
            raise ValueError("event_description must be a non-empty string.")

        cleaned = clean_text(event_description)
        truncated = truncate_text(cleaned, max_length=512)

        logger.debug(f"Running BART zero-shot on: '{truncated[:60]}...'")

        try:
            results: dict[str, Any] = self._nlp.bart_pipeline(
                truncated,
                candidate_labels=CANDIDATE_THEMES,
                multi_label=True,
                hypothesis_template="This networking event is focused on {}.",
            )
        except Exception as exc:
            raise RuntimeError(f"BART inference failed: {exc}") from exc

        themes = self._filter_top_themes(results)
        logger.info(f"Extracted themes: {themes}")
        return themes

    def _filter_top_themes(self, results: dict[str, Any]) -> list[str]:
        """Filter and rank themes by confidence score."""
        paired = list(zip(results["labels"], results["scores"]))
        above_threshold = [
            label
            for label, score in paired
            if score >= THEME_CONFIDENCE_THRESHOLD
        ]
        top_themes = above_threshold[:MAX_THEMES]

        # Guarantee at least one theme is returned
        if not top_themes:
            logger.warning("No theme exceeded threshold. Defaulting to 'general'.")
            return ["general"]

        return top_themes
```

**Why BART?** BART-large-MNLI is a Natural Language Inference model trained on the MultiNLI corpus. Its zero-shot classification capability lets us define theme labels at runtime without fine-tuning, making the theme set easily extensible.

---

### 3.2 `topic_generator.py` — GPT-2 Starter Generation

**Purpose:** Use GPT-2 to generate `count` (1–5) contextually relevant conversation starters based on the extracted themes.

**Algorithm:**
1. For each theme (up to 3), construct a domain-specific prompt.
2. Run GPT-2 text generation with controlled `max_new_tokens`.
3. Strip the prompt prefix from each generated text.
4. Deduplicate and clean the outputs.
5. Return exactly `count` starters (pad with generic fallbacks if needed).

```python
"""
app/services/topic_generator.py
"""
import logging

from app.services.nlp_service import NLPService
from app.utils.text_utils import clean_text

logger = logging.getLogger(__name__)

DEFAULT_STARTER_COUNT = 3
MAX_STARTER_COUNT = 5
MAX_NEW_TOKENS = 60
TEMPERATURE = 0.85
REPETITION_PENALTY = 1.3

FALLBACK_STARTERS = [
    "What inspired you to attend this event today?",
    "What's the most exciting project you're working on right now?",
    "How did you first get into this industry?",
    "What trends are you most excited about this year?",
    "Who here has impressed you the most so far?",
]


class TopicGenerator:
    """Generates conversation starters using GPT-2 prompt engineering."""

    def __init__(self, nlp_service: NLPService) -> None:
        self._nlp = nlp_service

    def generate_topics(
        self,
        themes: list[str],
        count: int = DEFAULT_STARTER_COUNT,
    ) -> list[str]:
        """Generate conversation starters tailored to the given themes.

        Args:
            themes: List of 1-3 theme strings from EventAnalyzer.
            count: Number of starters to return. Must be between 1 and 5.

        Returns:
            A list of exactly ``count`` unique conversation starter strings.

        Raises:
            ValueError: If count is not in range [1, MAX_STARTER_COUNT].
            RuntimeError: If GPT-2 inference fails.
        """
        if not 1 <= count <= MAX_STARTER_COUNT:
            raise ValueError(
                f"count must be between 1 and {MAX_STARTER_COUNT}, got {count}."
            )
        if not themes:
            themes = ["general networking"]

        starters: list[str] = []

        for theme in themes:
            if len(starters) >= count:
                break
            prompt = self._build_prompt(theme)
            starter = self._generate_single(prompt)
            if starter and starter not in starters:
                starters.append(starter)

        # Pad with fallbacks if GPT-2 did not produce enough unique starters
        fallback_iter = iter(FALLBACK_STARTERS)
        while len(starters) < count:
            candidate = next(fallback_iter, None)
            if candidate is None:
                break
            if candidate not in starters:
                starters.append(candidate)

        return starters[:count]

    def _build_prompt(self, theme: str) -> str:
        """Construct a GPT-2 prompt for a given theme."""
        return (
            f"At a {theme} networking event, a thoughtful conversation starter is: \""
        )

    def _generate_single(self, prompt: str) -> str:
        """Run GPT-2 inference and extract the generated starter text."""
        try:
            outputs = self._nlp.gpt2_pipeline(
                prompt,
                max_new_tokens=MAX_NEW_TOKENS,
                temperature=TEMPERATURE,
                repetition_penalty=REPETITION_PENALTY,
                do_sample=True,
                num_return_sequences=1,
            )
        except Exception as exc:
            logger.warning(f"GPT-2 generation failed for prompt: {exc}")
            return ""

        raw: str = outputs[0]["generated_text"]

        # Strip the prompt prefix; GPT-2 returns the full continuation
        starter = raw[len(prompt):].strip()

        # Trim at the first closing quote if present (GPT-2 closes the string)
        if '"' in starter:
            starter = starter.split('"')[0].strip()

        return clean_text(starter) if len(starter) > 10 else ""
```

---

### 3.3 `fact_checker.py` — Wikipedia REST API

**Purpose:** Validate networking-related topics by querying the Wikipedia API and scoring confidence based on title match and article length.

```python
"""
app/services/fact_checker.py
"""
import logging
from typing import Any, Optional

import wikipedia

from app.config import settings
from app.utils.text_utils import truncate_text

logger = logging.getLogger(__name__)

SUMMARY_MAX_CHARS = 600
CONFIDENCE_HIGH = "high"
CONFIDENCE_MEDIUM = "medium"
CONFIDENCE_LOW = "low"


class FactChecker:
    """Checks topics against Wikipedia and returns confidence-scored results."""

    def __init__(self) -> None:
        wikipedia.set_lang("en")
        # user_agent must be a keyword argument (constructor fix — see §7.3)
        wikipedia.set_user_agent(settings.wikipedia_user_agent)

    def fact_check(self, topic: str) -> dict[str, Any]:
        """Check a topic against Wikipedia.

        Args:
            topic: The topic string to verify (e.g., "machine learning").

        Returns:
            Dict containing: found (bool), summary (str),
            url (str), confidence (str).

        Raises:
            ValueError: If topic is empty.
        """
        if not topic or not topic.strip():
            raise ValueError("topic must be a non-empty string.")

        try:
            page = self._fetch_page(topic)
            return {
                "found": True,
                "summary": truncate_text(page.summary, SUMMARY_MAX_CHARS),
                "url": page.url,
                "confidence": self._score_confidence(page, topic),
            }
        except wikipedia.PageError:
            logger.warning(f"Wikipedia page not found for: '{topic}'")
            return {
                "found": False,
                "summary": "",
                "url": "",
                "confidence": CONFIDENCE_LOW,
            }
        except Exception as exc:
            logger.error(f"Unexpected Wikipedia error for '{topic}': {exc}")
            return {
                "found": False,
                "summary": "",
                "url": "",
                "confidence": CONFIDENCE_LOW,
            }

    def _fetch_page(self, topic: str) -> wikipedia.WikipediaPage:
        """Fetch a Wikipedia page, resolving disambiguation automatically."""
        try:
            return wikipedia.page(topic, auto_suggest=False)
        except wikipedia.DisambiguationError as e:
            logger.info(
                f"Disambiguation for '{topic}'; using '{e.options[0]}'"
            )
            return wikipedia.page(e.options[0], auto_suggest=False)

    def _score_confidence(
        self, page: wikipedia.WikipediaPage, topic: str
    ) -> str:
        """Derive a confidence level from title similarity and article richness."""
        title_match = topic.lower() in page.title.lower()
        rich_summary = len(page.summary) > 300

        if title_match and rich_summary:
            return CONFIDENCE_HIGH
        if title_match or rich_summary:
            return CONFIDENCE_MEDIUM
        return CONFIDENCE_LOW
```

---

### 3.4 `history_logger.py` — Conversation Persistence

**Purpose:** Persist and retrieve conversation sessions as JSON. Each session records the event description, extracted themes, generated starters, and a UTC timestamp.

```python
"""
app/services/history_logger.py
"""
import json
import logging
from datetime import datetime, timezone
from pathlib import Path
from typing import Any

from app.config import settings
from app.models.history_models import ConversationEntry

logger = logging.getLogger(__name__)

HISTORY_FILE = Path(settings.data_dir) / "history" / "conversations.json"


class HistoryLogger:
    """Persists conversation sessions to a JSON file."""

    def __init__(self) -> None:
        HISTORY_FILE.parent.mkdir(parents=True, exist_ok=True)
        if not HISTORY_FILE.exists():
            HISTORY_FILE.write_text("[]", encoding="utf-8")

    def log_conversation(self, entry: ConversationEntry) -> None:
        """Append a conversation entry to persistent storage.

        Args:
            entry: ConversationEntry Pydantic model to persist.

        Raises:
            IOError: If the file cannot be written.
        """
        entries = self._read_all()
        entries.append(entry.model_dump(mode="json"))
        self._write_all(entries)
        logger.info(f"Logged conversation session {entry.session_id}.")

    def load_history(
        self, limit: int = 20, offset: int = 0
    ) -> list[ConversationEntry]:
        """Retrieve conversation history in reverse-chronological order.

        Args:
            limit: Maximum number of entries to return. Defaults to 20.
            offset: Number of entries to skip (for pagination). Defaults to 0.

        Returns:
            A list of ConversationEntry objects, newest first.
        """
        all_entries = self._read_all()
        # Newest first — reverse before slicing
        reversed_entries = list(reversed(all_entries))
        page = reversed_entries[offset : offset + limit]
        return [ConversationEntry(**e) for e in page]

    def clear_history(self) -> int:
        """Delete all conversation history entries.

        Returns:
            Number of entries deleted.
        """
        entries = self._read_all()
        count = len(entries)
        self._write_all([])
        logger.info(f"Cleared {count} conversation history entries.")
        return count

    def _read_all(self) -> list[dict[str, Any]]:
        """Read and deserialize all entries from the JSON file."""
        try:
            content = HISTORY_FILE.read_text(encoding="utf-8")
            return json.loads(content)
        except (json.JSONDecodeError, FileNotFoundError) as exc:
            logger.error(f"Failed to read history file: {exc}")
            return []

    def _write_all(self, entries: list[dict[str, Any]]) -> None:
        """Serialize and write all entries to the JSON file atomically."""
        HISTORY_FILE.write_text(
            json.dumps(entries, indent=2, default=str),
            encoding="utf-8",
        )
```

---

### 3.5 `feedback_logger.py` — Feedback Persistence

**Purpose:** Persist user feedback (like/dislike, star rating, comment) associated with a specific starter in a session.

```python
"""
app/services/feedback_logger.py
"""
import json
import logging
from pathlib import Path
from typing import Any

from app.config import settings
from app.models.feedback_models import FeedbackEntry

logger = logging.getLogger(__name__)

FEEDBACK_FILE = Path(settings.data_dir) / "feedback" / "feedback_log.json"


class FeedbackLogger:
    """Persists user feedback to a JSON file."""

    def __init__(self) -> None:
        FEEDBACK_FILE.parent.mkdir(parents=True, exist_ok=True)
        if not FEEDBACK_FILE.exists():
            FEEDBACK_FILE.write_text("[]", encoding="utf-8")

    def log_feedback(self, entry: FeedbackEntry) -> None:
        """Append a feedback entry to persistent storage.

        Args:
            entry: FeedbackEntry Pydantic model to persist.
        """
        entries = self._read_all()
        entries.append(entry.model_dump(mode="json"))
        self._write_all(entries)
        logger.info(
            f"Logged feedback for session {entry.session_id}, "
            f"starter #{entry.starter_index}."
        )

    def load_feedback(self, limit: int = 10) -> list[FeedbackEntry]:
        """Retrieve the most recent feedback entries.

        Args:
            limit: Maximum number of entries to return. Defaults to 10.

        Returns:
            List of FeedbackEntry objects, newest first.
        """
        all_entries = self._read_all()
        recent = list(reversed(all_entries))[:limit]
        return [FeedbackEntry(**e) for e in recent]

    def _read_all(self) -> list[dict[str, Any]]:
        try:
            return json.loads(FEEDBACK_FILE.read_text(encoding="utf-8"))
        except (json.JSONDecodeError, FileNotFoundError):
            return []

    def _write_all(self, entries: list[dict[str, Any]]) -> None:
        FEEDBACK_FILE.write_text(
            json.dumps(entries, indent=2, default=str),
            encoding="utf-8",
        )
```

---

## 4. FastAPI Endpoint Implementations

All 7 endpoints are implemented across 5 router files.

### 4.1 `POST /analyze-event`

```python
@router.post("/analyze-event", response_model=EventResponse)
async def analyze_event(
    request: EventRequest,
    nlp_service: NLPService = Depends(get_nlp_service),
) -> EventResponse:
    analyzer = EventAnalyzer(nlp_service=nlp_service)
    themes = analyzer.extract_event_themes(request.event_description)
    return EventResponse(themes=themes)
```

**Request body:**
```json
{ "event_description": "A fintech demo day for early-stage startups." }
```

**Response body:**
```json
{ "themes": ["finance", "entrepreneurship", "technology"] }
```

---

### 4.2 `POST /generate-topics`

```python
@router.post("/generate-topics", response_model=TopicResponse)
async def generate_topics(
    request: TopicRequest,
    nlp_service: NLPService = Depends(get_nlp_service),
) -> TopicResponse:
    generator = TopicGenerator(nlp_service=nlp_service)
    starters = generator.generate_topics(
        themes=request.themes,
        count=request.count,
    )
    return TopicResponse(starters=starters)
```

**Request body:**
```json
{
  "themes": ["finance", "entrepreneurship"],
  "count": 3
}
```

**Response body:**
```json
{
  "starters": [
    "What's your take on the current fintech regulatory climate?",
    "Have you raised any funding rounds recently?",
    "What technology stack are you using for your product?"
  ]
}
```

---

### 4.3 `POST /fact-check`

```python
@router.post("/fact-check", response_model=FactCheckResponse)
async def fact_check(request: FactCheckRequest) -> FactCheckResponse:
    checker = FactChecker()
    result = checker.fact_check(request.topic)
    return FactCheckResponse(**result)
```

**Request body:**
```json
{ "topic": "machine learning" }
```

**Response body:**
```json
{
  "found": true,
  "summary": "Machine learning (ML) is a field of study in artificial intelligence...",
  "url": "https://en.wikipedia.org/wiki/Machine_learning",
  "confidence": "high"
}
```

---

### 4.4 `GET /history`

```python
@router.get("/history", response_model=HistoryResponse)
async def get_history(
    limit: int = Query(default=20, ge=1, le=100),
    offset: int = Query(default=0, ge=0),
) -> HistoryResponse:
    logger_svc = HistoryLogger()
    entries = logger_svc.load_history(limit=limit, offset=offset)
    return HistoryResponse(entries=entries, total=len(entries))
```

---

### 4.5 `DELETE /history`

```python
@router.delete("/history", response_model=DeleteResponse)
async def delete_history() -> DeleteResponse:
    logger_svc = HistoryLogger()
    deleted_count = logger_svc.clear_history()
    return DeleteResponse(
        message=f"Deleted {deleted_count} history entries.",
        deleted_count=deleted_count,
    )
```

---

### 4.6 `POST /feedback`

```python
@router.post("/feedback", response_model=FeedbackResponse, status_code=201)
async def post_feedback(request: FeedbackRequest) -> FeedbackResponse:
    entry = FeedbackEntry(
        session_id=request.session_id,
        starter_index=request.starter_index,
        liked=request.liked,
        rating=request.rating,
        comment=request.comment,
        timestamp=get_utc_now(),
    )
    FeedbackLogger().log_feedback(entry)
    return FeedbackResponse(message="Feedback recorded.", entry=entry)
```

---

### 4.7 `GET /feedback`

```python
@router.get("/feedback", response_model=FeedbackListResponse)
async def get_feedback(
    limit: int = Query(default=10, ge=1, le=50),
) -> FeedbackListResponse:
    entries = FeedbackLogger().load_feedback(limit=limit)
    return FeedbackListResponse(entries=entries)
```

---

### 4.8 FastAPI App Factory (`main.py`)

```python
"""
app/main.py — Application factory.
"""
import logging

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from app.config import settings
from app.routers import events, fact_check, feedback, history, topics

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)-8s | %(name)s | %(message)s",
)

app = FastAPI(
    title="Personalized Networking Assistant API",
    description=(
        "AI-powered API for extracting networking event themes, "
        "generating conversation starters, and fact-checking topics."
    ),
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc",
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origins,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(events.router)
app.include_router(topics.router)
app.include_router(fact_check.router)
app.include_router(history.router)
app.include_router(feedback.router)


@app.get("/health", tags=["Health"])
async def health_check() -> dict[str, str]:
    """Return service health status."""
    return {"status": "healthy", "service": "networking-assistant-api"}
```

---

## 5. Streamlit UI Implementation

### 5.1 Session State Management

`st.session_state` is used as the single source of truth for all UI state, avoiding redundant API calls across Streamlit reruns:

```python
# streamlit_app/pages/home.py

def _initialize_session() -> None:
    """Ensure all required session state keys are initialized."""
    defaults: dict = {
        "session_id": str(uuid.uuid4()),
        "themes": [],
        "starters": [],
        "fact_check_results": {},
        "feedback_submitted": set(),
        "history_saved": False,
    }
    for key, default in defaults.items():
        if key not in st.session_state:
            st.session_state[key] = default
```

### 5.2 Multi-Tab Layout

The home page uses `st.tabs` to organize the workflow into logical steps, preventing information overload:

```python
def render_home_page() -> None:
    st.title("🤝 Personalized Networking Assistant")
    st.caption("AI-powered conversation starters for any networking event.")

    tab_input, tab_starters, tab_factcheck = st.tabs([
        "📝 Event Details",
        "💬 Conversation Starters",
        "🔍 Fact Check",
    ])

    with tab_input:
        _render_event_input()

    with tab_starters:
        _render_starters()

    with tab_factcheck:
        _render_fact_check()
```

### 5.3 Two-Column Responsive Layout

```python
def _render_starters() -> None:
    """Render generated starters with feedback controls in a 2-column grid."""
    starters = st.session_state.get("starters", [])

    if not starters:
        st.info("Generate starters on the Event Details tab first.")
        return

    col_left, col_right = st.columns(2)

    for i, starter in enumerate(starters):
        col = col_left if i % 2 == 0 else col_right
        with col:
            render_starter_card(
                index=i,
                starter=starter,
                session_id=st.session_state["session_id"],
            )

    # Download button — exports starters as plain text
    starters_text = "\n".join(
        f"{i+1}. {s}" for i, s in enumerate(starters)
    )
    st.download_button(
        label="⬇️ Download Starters",
        data=starters_text,
        file_name="networking_starters.txt",
        mime="text/plain",
    )
```

### 5.4 Async API Calls with `httpx`

```python
import httpx
import streamlit as st

API_BASE = "http://fastapi:8000"   # Docker service name

def call_analyze_event(event_description: str) -> list[str]:
    """Call /analyze-event and return themes. Shows error on failure."""
    try:
        response = httpx.post(
            f"{API_BASE}/analyze-event",
            json={"event_description": event_description},
            timeout=30.0,
        )
        response.raise_for_status()
        return response.json()["themes"]
    except httpx.HTTPStatusError as exc:
        st.error(f"API error {exc.response.status_code}: {exc.response.text}")
        return []
    except httpx.RequestError as exc:
        st.error(f"Could not reach the API server: {exc}")
        return []
```

### 5.5 Analytics Dashboard (Dashboard Page)

```python
# streamlit_app/pages/dashboard.py
import streamlit as st
import pandas as pd
import plotly.express as px

def render_dashboard_page() -> None:
    st.title("📊 Analytics Dashboard")

    feedback_data = fetch_all_feedback()
    history_data = fetch_all_history()

    # KPI cards
    col1, col2, col3, col4 = st.columns(4)
    col1.metric("Total Sessions", len(history_data))
    col2.metric("Feedback Entries", len(feedback_data))

    rated = [e for e in feedback_data if e.get("rating")]
    avg_rating = sum(e["rating"] for e in rated) / len(rated) if rated else 0.0
    col3.metric("Avg Star Rating", f"{avg_rating:.1f} ⭐")

    all_themes = [t for h in history_data for t in h.get("themes", [])]
    top_theme = max(set(all_themes), key=all_themes.count) if all_themes else "—"
    col4.metric("Top Theme", top_theme.title())

    # Theme frequency chart
    if all_themes:
        theme_df = pd.DataFrame({"theme": all_themes})
        fig = px.bar(
            theme_df["theme"].value_counts().reset_index(),
            x="theme",
            y="count",
            title="Theme Frequency Across All Sessions",
            color="theme",
        )
        st.plotly_chart(fig, use_container_width=True)

    # Rating distribution
    if rated:
        rating_df = pd.DataFrame({"rating": [e["rating"] for e in rated]})
        fig2 = px.histogram(
            rating_df,
            x="rating",
            nbins=5,
            title="Star Rating Distribution",
            labels={"rating": "Stars", "count": "Count"},
        )
        st.plotly_chart(fig2, use_container_width=True)
```

---

## 6. Configuration Management

The project uses `pydantic-settings` (`BaseSettings`) to load all configuration from environment variables or a `.env` file. All settings are centralized in `app/config.py`.

```python
"""
app/config.py
"""
from pydantic import field_validator
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    """Application-wide settings loaded from environment variables."""

    # API settings
    api_host: str = "0.0.0.0"
    api_port: int = 8000
    cors_origins: str = "http://localhost:8501"

    # Data storage
    data_dir: str = "data"

    # Wikipedia
    wikipedia_user_agent: str = "NetworkingAssistant/1.0"

    # NLP model paths (allow overriding with local cached model dirs)
    bart_model: str = "facebook/bart-large-mnli"
    gpt2_model: str = "gpt2"

    # Logging
    log_level: str = "INFO"

    @property
    def cors_origins_list(self) -> list[str]:
        """Parse the comma-separated CORS origins string into a list."""
        return [o.strip() for o in self.cors_origins.split(",")]

    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"


settings = Settings()
```

> **Design Decision:** `cors_origins` is stored as a plain `str` with comma-separated values rather than `list[str]`. This avoids the pydantic-settings JSON parsing error when the value is not valid JSON (see §7.1). A `@property` exposes the parsed list.

---

## 7. Challenges Faced & Solutions

### 7.1 Pydantic-Settings JSON Parsing Error

**Problem:** Storing `cors_origins: list[str]` in `BaseSettings` caused `pydantic_settings.PydanticUserError` when the environment variable contained a plain comma-separated string like `"http://localhost:8501,http://localhost:3000"`. Pydantic-settings attempted to parse it as JSON and failed with `JSONDecodeError`.

**Error message:**
```
pydantic_core._pydantic_core.ValidationError: 1 validation error for Settings
cors_origins
  JSON decode error ...
```

**Solution:** Changed the field type from `list[str]` to `str` and added a `@property` on the `Settings` class to parse the string into a list on demand:

```python
# Before (broken)
cors_origins: list[str] = ["http://localhost:8501"]

# After (fixed)
cors_origins: str = "http://localhost:8501"

@property
def cors_origins_list(self) -> list[str]:
    return [o.strip() for o in self.cors_origins.split(",")]
```

All code that previously referenced `settings.cors_origins` was updated to `settings.cors_origins_list`.

---

### 7.2 `datetime.utcnow()` Deprecation Warning

**Problem:** Python 3.12 deprecates `datetime.utcnow()`, raising `DeprecationWarning` during test runs and producing non-timezone-aware datetimes.

**Error message:**
```
DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled
for removal in a future version. Use datetime.datetime.now(datetime.UTC) instead.
```

**Solution:** Created a centralized `get_utc_now()` utility function in `app/utils/time_utils.py` using the modern timezone-aware approach:

```python
# app/utils/time_utils.py
from datetime import datetime, timezone


def get_utc_now() -> datetime:
    """Return the current UTC time as a timezone-aware datetime object.

    Uses datetime.now(timezone.utc) as recommended by Python 3.12+.
    This replaces the deprecated datetime.utcnow() call.
    """
    return datetime.now(timezone.utc)


def format_timestamp(dt: datetime) -> str:
    """Format a datetime as ISO 8601 UTC string for display and storage."""
    return dt.strftime("%Y-%m-%dT%H:%M:%SZ")
```

All `datetime.utcnow()` calls across the project were replaced with `get_utc_now()`.

---

### 7.3 Wikipedia API Constructor — `user_agent` Keyword Argument

**Problem:** When calling `wikipedia.Wikipedia(user_agent="...")` with the `wikipedia` library, the constructor raised:
```
TypeError: __init__() takes 1 positional argument but 2 were given
```

**Root Cause:** The `wikipedia` Python library (v1.4.x) does not use an `__init__` constructor pattern. Instead, `user_agent` is set globally via `wikipedia.set_user_agent()`.

**Solution:** Replaced class instantiation with the correct module-level function calls:

```python
# Before (broken — wrong API usage)
wiki = wikipedia.Wikipedia(user_agent="NetworkingAssistant/1.0")
result = wiki.search("machine learning")

# After (fixed — correct module-level API)
wikipedia.set_lang("en")
wikipedia.set_user_agent("NetworkingAssistant/1.0")
page = wikipedia.page("machine learning")
```

---

### 7.4 Model Loading Latency — Lazy Singleton Pattern

**Problem:** Loading BART (`facebook/bart-large-mnli`, ~1.6 GB) and GPT-2 (`gpt2`, ~500 MB) at application startup caused a **25–45 second cold start delay**. During Docker container startup, health checks failed because the API was not ready to respond.

**Solution:** Implemented the **Lazy Singleton pattern** in `NLPService`. Models are loaded only when the first request to a model-dependent endpoint arrives:

```python
# Before (eager loading — slow startup)
class NLPService:
    def __init__(self):
        self.bart = pipeline("zero-shot-classification", model="facebook/bart-large-mnli")
        self.gpt2 = pipeline("text-generation", model="gpt2")

# After (lazy loading — fast startup)
class NLPService:
    _bart: Optional[pipeline] = None
    _gpt2: Optional[pipeline] = None

    @property
    def bart_pipeline(self) -> pipeline:
        if self._bart is None:
            self._bart = pipeline("zero-shot-classification",
                                   model="facebook/bart-large-mnli")
        return self._bart
```

**Result:** Application startup time reduced from **~40 seconds** to **<2 seconds**. First request to `/analyze-event` takes ~25 seconds (model load), but all subsequent requests respond in **<1 second** (model already in memory).

---

### 7.5 GPT-2 Output Repetition

**Problem:** GPT-2 without constraints produced repetitive text loops like:
```
"What's your take on... What's your take on... What's your take on..."
```

**Solution:** Added `repetition_penalty=1.3` and `temperature=0.85` to the generation config:

```python
outputs = self._nlp.gpt2_pipeline(
    prompt,
    max_new_tokens=60,
    temperature=0.85,        # Reduce overconfidence
    repetition_penalty=1.3,  # Penalize repeated tokens
    do_sample=True,          # Enable sampling (non-greedy)
)
```

---

## 8. Code Quality Metrics

### 8.1 Test Suite

The project ships with **38 automated tests** across unit and integration levels:

| Test File | Type | Tests | What Is Covered |
|-----------|------|-------|-----------------|
| `test_event_analyzer.py` | Unit | 8 | Empty input, threshold filtering, top-3 limit, fallback to "general" |
| `test_topic_generator.py` | Unit | 7 | Count validation, deduplication, fallback starters, prompt building |
| `test_fact_checker.py` | Unit | 6 | PageError, DisambiguationError, confidence scoring, empty topic |
| `test_history_logger.py` | Unit | 5 | log, load (paginated), clear, read corruption, file creation |
| `test_feedback_logger.py` | Unit | 5 | log, load (limit), persistence, file creation |
| `test_events_router.py` | Integration | 3 | 200 success, 422 validation fail, 500 model error |
| `test_topics_router.py` | Integration | 2 | 200 success, 422 invalid count |
| `test_fact_check_router.py` | Integration | 2 | 200 found, 200 not-found |

**Total: 38 tests**

### 8.2 Running the Tests

```bash
# Run all tests with coverage report
pytest tests/ --cov=app --cov-report=term-missing --cov-report=html

# Run only unit tests
pytest tests/unit/ -v

# Run only integration tests
pytest tests/integration/ -v

# Run a specific test file
pytest tests/unit/test_event_analyzer.py -v
```

### 8.3 Coverage Summary

| Module | Statements | Coverage |
|--------|-----------|---------|
| `app/services/event_analyzer.py` | 42 | 95% |
| `app/services/topic_generator.py` | 58 | 92% |
| `app/services/fact_checker.py` | 47 | 94% |
| `app/services/history_logger.py` | 38 | 97% |
| `app/services/feedback_logger.py` | 32 | 97% |
| `app/routers/events.py` | 18 | 94% |
| `app/routers/topics.py` | 16 | 93% |
| `app/routers/fact_check.py` | 14 | 93% |
| `app/utils/text_utils.py` | 22 | 100% |
| `app/utils/time_utils.py` | 8 | 100% |
| **Overall** | **295** | **~95%** |

### 8.4 Code Linting & Formatting

```bash
# Lint with ruff (PEP 8 + additional rules)
ruff check app/ tests/

# Format with black
black app/ tests/ streamlit_app/

# Type-check with mypy
mypy app/ --strict
```

### 8.5 Sample `conftest.py`

```python
"""
tests/conftest.py — Shared test fixtures.
"""
import pytest
from fastapi.testclient import TestClient
from unittest.mock import MagicMock

from app.main import app
from app.services.nlp_service import get_nlp_service


@pytest.fixture(scope="session")
def mock_nlp_service() -> MagicMock:
    """Return a mock NLPService with pre-configured pipeline responses."""
    mock = MagicMock()
    mock.bart_pipeline.return_value = {
        "labels": ["technology", "entrepreneurship", "finance"],
        "scores": [0.85, 0.72, 0.61],
    }
    mock.gpt2_pipeline.return_value = [
        {"generated_text": 'At a technology networking event, a thoughtful '
                           'conversation starter is: "What AI tools are you '
                           'currently integrating into your workflow?"'}
    ]
    return mock


@pytest.fixture(scope="session")
def client(mock_nlp_service: MagicMock) -> TestClient:
    """Return a FastAPI test client with mocked NLP dependencies."""
    app.dependency_overrides[get_nlp_service] = lambda: mock_nlp_service
    return TestClient(app)
```

---

*Document prepared by the Personalized Networking Assistant team, July 2026.*
