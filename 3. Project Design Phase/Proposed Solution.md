# Proposed Solution
## Personalized Networking Assistant

---

> **Project:** Personalized Networking Assistant  
> **Phase:** 3 — Project Design  
> **Document Type:** Proposed Solution  
> **Developer:** mehboob ali mohammed  
> **Team Members:** mehboob ali mohammed  
> **Date:** July 2026  
> **Version:** 1.0

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Core Solution Components](#2-core-solution-components)
3. [BART Zero-Shot Classification — Theme Extraction](#3-bart-zero-shot-classification--theme-extraction)
4. [GPT-2 Small — Conversation Generation](#4-gpt-2-small--conversation-generation)
5. [Wikipedia REST API — Fact-Checking](#5-wikipedia-rest-api--fact-checking)
6. [Local JSON Persistence — History and Feedback](#6-local-json-persistence--history-and-feedback)
7. [Streamlit Dark-Themed UI](#7-streamlit-dark-themed-ui)
8. [User Flow — End-to-End](#8-user-flow--end-to-end)
9. [System Boundaries and Interfaces](#9-system-boundaries-and-interfaces)
10. [Ethical AI Considerations](#10-ethical-ai-considerations)
11. [Expected Outcomes and Impact](#11-expected-outcomes-and-impact)

---

## 1. Executive Summary

### What the Solution Is

The **Personalized Networking Assistant** is a full-stack, AI-powered web application that generates personalized, contextually grounded, and fact-verified conversation starters for professional networking events. The system integrates two transformer-based NLP models, a live fact-checking API, and a structured feedback mechanism within a polished multi-page user interface.

### What Problem It Solves

The solution targets a well-documented gap in professional networking tools: the inability to generate conversation starters that are simultaneously **contextual** (event-aware), **personalized** (user-role-aware), and **verified** (factually accurate). Existing tools provide generic templates. This application provides intelligent, individualized output grounded in real-world knowledge.

### How It Solves It — In Brief

```
Input: Event name + User role + User interests
           ↓
   [BART] Theme Extraction
           ↓
   [Prompt Builder] Personalized prompt assembly
           ↓
   [GPT-2] Conversation starter generation
           ↓
   [Wikipedia API] Fact-checking + confidence annotation
           ↓
   [Streamlit UI] Display with Like/Dislike controls
           ↓
   [JSON Storage] History + Feedback persistence
```

### Who Built It

The solution is designed and implemented by a four-person team:

| Name | Role |
|---|---|
| Shaik Sumiya Zainab | Team Lead — Architecture, Integration, Documentation |
| Naga Jagan Mohan Rao Thattukolla | NLP Backend — BART, FastAPI endpoints |
| Satvika Tallam | NLP Backend — GPT-2, Prompt Engineering, UI |
| Tejesh Velivela | Fact-Check Service, Feedback System, Data Layer |

### Technology Stack Summary

```
Frontend:   Streamlit 1.x (Python, dark theme, multi-page)
Backend:    FastAPI (Python 3.10+, Uvicorn ASGI)
Model 1:    facebook/bart-large-mnli (HuggingFace Transformers)
Model 2:    gpt2 (HuggingFace Transformers)
Fact-Check: Wikipedia REST API v1 (no auth)
Storage:    JSON files (history.json, feedback.json)
Config:     python-dotenv (.env file)
Container:  Docker + docker-compose
```

---

## 2. Core Solution Components

The solution is composed of five primary components that operate in a coordinated pipeline. Each component has a single, well-defined responsibility.

---

### Component Map (Text Diagram)

```
┌─────────────────────────────────────────────────────────────────────┐
│                     PERSONALIZED NETWORKING ASSISTANT               │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                 PRESENTATION LAYER (Streamlit)               │   │
│  │   ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐ │   │
│  │   │ Generator Tab│  │ History Tab  │  │  Feedback Tab    │ │   │
│  │   │  (main UI)   │  │ (past runs)  │  │  (analytics)     │ │   │
│  │   └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘ │   │
│  └──────────┼─────────────────┼────────────────────┼───────────┘   │
│             │  HTTP/REST       │                    │               │
│  ┌──────────▼─────────────────▼────────────────────▼───────────┐   │
│  │                   API LAYER (FastAPI)                        │   │
│  │  /classify  /generate  /factcheck  /history  /feedback  ... │   │
│  └──────┬──────────┬──────────┬──────────┬───────────┬─────────┘   │
│         │          │          │          │           │               │
│  ┌──────▼──┐ ┌─────▼───┐ ┌───▼─────┐ ┌─▼────────┐ ┌▼──────────┐  │
│  │  BART   │ │  GPT-2  │ │Wikipedia│ │ History  │ │ Feedback  │  │
│  │Service  │ │ Service │ │ Service │ │ Service  │ │ Service   │  │
│  └──────┬──┘ └─────┬───┘ └───┬─────┘ └─┬────────┘ └┬──────────┘  │
│         │          │          │          │            │              │
│  ┌──────▼──────────▼──────────▼──────────▼────────────▼──────────┐ │
│  │                      DATA LAYER                                │ │
│  │         history.json              feedback.json                │ │
│  └───────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

---

### Component 1 — Theme Extraction (BART)

- **What it does:** Analyzes the user-provided event name and description to identify 2–3 dominant professional themes.
- **Input:** Raw event name string.
- **Output:** Ordered list of thematic labels with confidence scores.
- **Module:** `services/bart_classifier.py`
- **Endpoint:** `POST /api/classify`

---

### Component 2 — Conversation Generation (GPT-2)

- **What it does:** Constructs a personalized prompt from user metadata and extracted themes, then uses GPT-2 to generate 3–5 conversation starter candidates.
- **Input:** User role, interests, extracted themes, event name.
- **Output:** List of natural-language conversation starters.
- **Module:** `services/gpt2_generator.py`
- **Endpoint:** `POST /api/generate`

---

### Component 3 — Fact-Checking (Wikipedia API)

- **What it does:** Scans generated starters for named entities, queries the Wikipedia REST API for each, and annotates each starter with a verification status.
- **Input:** List of generated starters.
- **Output:** Same list annotated with `fact_check_status` ∈ `{verified, unverified, warning}`.
- **Module:** `services/fact_checker.py`
- **Endpoint:** `POST /api/factcheck`

---

### Component 4 — History Persistence (JSON)

- **What it does:** Automatically saves each generation session (inputs, themes, starters, fact-check results) to a local JSON file. Exposes retrieval and filtering endpoints.
- **Input:** Session data object.
- **Output:** Session ID; retrieval of all or filtered history.
- **Module:** `services/history_service.py`
- **Endpoint:** `POST /api/history/save`, `GET /api/history`

---

### Component 5 — Feedback System (JSON)

- **What it does:** Records per-starter Like/Dislike feedback from the user, keyed by starter ID and session ID.
- **Input:** Starter ID, session ID, sentiment (`like`/`dislike`).
- **Output:** Acknowledgement; retrieval of feedback analytics.
- **Module:** `services/feedback_service.py`
- **Endpoint:** `POST /api/feedback`, `GET /api/feedback`

---

## 3. BART Zero-Shot Classification — Theme Extraction

### Why BART Zero-Shot Was Chosen

The core requirement of Component 1 is to extract professional themes from an arbitrary event title without having any labeled training data for events. This rules out supervised classifiers. The solution requires a model that understands semantic meaning across professional domains generatively — a task for which **zero-shot classification is ideally suited**.

**Evaluation of Alternatives:**

| Approach | Pros | Cons | Selected? |
|---|---|---|---|
| Keyword matching (regex/TF-IDF) | Fast, no model | Brittle, misses semantics | ❌ |
| LDA / Topic Modeling | Unsupervised | Requires corpus, poor on short text | ❌ |
| Fine-tuned BERT classifier | High accuracy if trained | Requires labeled event data | ❌ |
| **BART zero-shot (MNLI)** | **No training needed, semantic** | Slower than keyword | ✅ |
| GPT-4 (API) | Very accurate | Costly, external API, privacy risk | ❌ |

**BART-large-mnli** was pre-trained on the Multi-Genre Natural Language Inference (MNLI) dataset, which teaches it to evaluate whether a premise entails a hypothesis. Zero-shot classification reframes label assignment as: *"Does this event description [premise] entail that it belongs to the domain [hypothesis]?"* This makes it remarkably effective for domain classification without any domain-specific training.

---

### How BART Zero-Shot Works in This System

#### Step 1 — Model Loading (Lazy, Cached)

```python
# services/bart_classifier.py

from transformers import pipeline
from functools import lru_cache

@lru_cache(maxsize=1)
def load_bart_pipeline():
    """Load BART zero-shot pipeline once; cache globally."""
    return pipeline(
        "zero-shot-classification",
        model="facebook/bart-large-mnli",
        device=-1  # CPU; set to 0 for GPU
    )
```

Model loading is deferred to first use and cached globally. This avoids re-loading the ~1.6GB model on every API call — critical for response latency.

#### Step 2 — Candidate Label Set

```python
CANDIDATE_LABELS = [
    "artificial intelligence",
    "machine learning",
    "data science",
    "software engineering",
    "cloud computing",
    "cybersecurity",
    "biotechnology",
    "healthcare technology",
    "finance and fintech",
    "sustainability and clean energy",
    "robotics and automation",
    "entrepreneurship and startups",
    "education technology",
    "agriculture technology",
    "human-computer interaction"
]
```

The candidate label set is designed to cover the major professional domains represented in modern networking events. Labels use natural language phrases rather than single keywords to maximize MNLI semantic matching.

#### Step 3 — Classification Call

```python
def classify_event_themes(event_name: str, top_k: int = 3) -> list[dict]:
    classifier = load_bart_pipeline()
    result = classifier(
        sequences=event_name,
        candidate_labels=CANDIDATE_LABELS,
        multi_label=True  # Event can belong to multiple domains
    )
    # Sort by score, return top_k
    sorted_labels = sorted(
        zip(result["labels"], result["scores"]),
        key=lambda x: x[1],
        reverse=True
    )
    return [
        {"theme": label, "confidence": round(score, 4)}
        for label, score in sorted_labels[:top_k]
    ]
```

**Key parameter — `multi_label=True`:** Most events span multiple domains (e.g., "AI in Healthcare" is both AI and Healthcare). Multi-label mode scores each label independently rather than forcing a single-winner classification, which is more accurate for event themes.

#### Step 4 — Output Example

```
Input:  "IEEE International Conference on AI-Driven Drug Discovery"

Output: [
  {"theme": "artificial intelligence",   "confidence": 0.9231},
  {"theme": "biotechnology",             "confidence": 0.8814},
  {"theme": "healthcare technology",     "confidence": 0.8402}
]
```

These three themes are then passed to the prompt builder for GPT-2.

---

### BART Performance Characteristics

| Metric | Value |
|---|---|
| Model size on disk | ~1.6 GB |
| First call latency (model load + inference) | 8–15 seconds |
| Subsequent call latency (model cached) | 1.5–3 seconds |
| Recommended hardware | CPU (4-core+) or GPU |
| Top-1 accuracy on 30-event test set (target) | ≥ 80% |

---

## 4. GPT-2 Small — Conversation Generation

### Why GPT-2 Small Was Chosen

GPT-2 Small is selected as the generation backbone for the following reasons:

1. **Local execution** — No API key, no cost, no data privacy concerns. All text is processed on the user's machine.
2. **Open weights** — Available via HuggingFace `transformers` with a single `from_pretrained("gpt2")` call.
3. **Controllable output** — Supports temperature, top-p, and repetition penalty for tunable generation quality.
4. **Sufficient capability for short-form generation** — Conversation starters are 1–3 sentences. GPT-2 Small (117M parameters) is well-suited for this length without the overhead of larger models.
5. **No fine-tuning required for MVP** — Prompt engineering alone produces acceptable results for the MVP scope.

**Comparison with alternatives:**

| Model | Size | Quality | Local? | Cost | Selected? |
|---|---|---|---|---|---|
| GPT-2 Small | 117M | Good (prompt-dependent) | ✅ | Free | ✅ |
| GPT-2 Medium | 345M | Better | ✅ | Free | (future) |
| Llama 2 7B | 7B | Excellent | ✅ | Free | Too large for MVP |
| GPT-4o (API) | Unknown | Excellent | ❌ | Paid | Privacy risk |
| Mistral 7B | 7B | Excellent | ✅ | Free | Too large for MVP |

---

### Prompt Engineering Strategy

Prompt engineering is the primary lever for improving GPT-2 quality without fine-tuning. The strategy is built around a **structured template** that provides maximum context with minimal ambiguity.

#### Master Prompt Template

```python
PROMPT_TEMPLATE = """You are an expert professional networking coach. 
Your task is to generate {num_starters} natural, specific, and open-ended 
conversation starters for a professional attending a networking event.

Attendee Profile:
- Professional Role: {user_role}
- Areas of Interest: {interests_str}
- Event Name: {event_name}
- Key Themes of the Event: {themes_str}

Instructions:
- Each starter should be a genuine, curiosity-driven question or observation.
- Avoid clichés like "What do you do?" or "Where are you from?"
- Make starters specific to the themes and the attendee's role.
- Starters should feel natural, not robotic or scripted.

Conversation Starters:
1."""
```

#### Template Variable Population

```python
def build_prompt(
    user_role: str,
    interests: list[str],
    event_name: str,
    themes: list[str],
    num_starters: int = 3
) -> str:
    return PROMPT_TEMPLATE.format(
        num_starters=num_starters,
        user_role=user_role,
        interests_str=", ".join(interests),
        event_name=event_name,
        themes_str=", ".join(themes),
    )
```

#### Generation Parameters

```python
def generate_starters(prompt: str, num_starters: int = 3) -> list[str]:
    tokenizer, model = load_gpt2()
    inputs = tokenizer(prompt, return_tensors="pt")
    
    outputs = model.generate(
        **inputs,
        max_new_tokens=250,         # Enough for 3 starters ~80 tokens each
        temperature=0.85,           # Moderate creativity
        top_p=0.92,                 # Nucleus sampling
        top_k=50,                   # Vocabulary pruning
        repetition_penalty=1.3,     # Discourages repetitive phrases
        do_sample=True,             # Enable probabilistic sampling
        num_return_sequences=1,     # Single generation pass
        pad_token_id=tokenizer.eos_token_id
    )
    
    generated_text = tokenizer.decode(
        outputs[0][inputs["input_ids"].shape[1]:],
        skip_special_tokens=True
    )
    
    return parse_starters(generated_text, num_starters)
```

#### Post-Processing — Starter Parsing

GPT-2 output is a continuous string. The parser extracts numbered starters:

```python
import re

def parse_starters(raw_text: str, num_starters: int) -> list[str]:
    """Extract numbered starters from raw GPT-2 output."""
    # Match lines like "1. ...", "2. ...", "3. ..."
    pattern = r'\d+\.\s+(.+?)(?=\d+\.|$)'
    matches = re.findall(pattern, raw_text, re.DOTALL)
    
    starters = []
    for match in matches[:num_starters]:
        cleaned = match.strip()
        # Ensure sentence ends with punctuation
        if cleaned and not cleaned[-1] in ".?!":
            cleaned += "?"
        # Filter out very short or clearly incomplete lines
        if len(cleaned) > 20:
            starters.append(cleaned)
    
    return starters if starters else [raw_text[:300]]  # Fallback
```

---

### GPT-2 Generation Example

**Input Context:**
```
User Role:    Senior Machine Learning Engineer
Interests:    MLOps, Kubernetes, model deployment
Event Name:   KubeCon North America 2026
Themes:       cloud computing, machine learning, software engineering
```

**Generated Starters (sample):**
```
1. "I've been exploring Kubernetes-native model serving with KServe — 
    are you seeing adoption of that pattern in production workloads at 
    your organization?"

2. "With so many teams moving toward GitOps for ML pipelines, what's 
    been your experience with Argo Workflows versus Kubeflow Pipelines 
    for orchestrating training runs?"

3. "I'm curious about the observability side — how are teams here 
    approaching drift detection and model monitoring at scale?"
```

---

### GPT-2 Performance Characteristics

| Metric | Value |
|---|---|
| Model size on disk | ~548 MB |
| First call latency (model load) | 3–6 seconds |
| Subsequent call latency | 4–8 seconds per generation |
| Output quality | Good for short-form, prompt-dependent |
| Hardware | CPU (4-core minimum) or GPU |

---

## 5. Wikipedia REST API — Fact-Checking

### Design Decision — Why Wikipedia REST API

The fact-checking requirement demands a real-time, authoritative information source that:
- Requires **no API key or authentication**.
- Has **broad coverage** of organizations, technologies, and events.
- Returns **machine-readable, structured data**.
- Is **free to use** without rate limit concerns for light usage.
- Is **privacy-preserving** — user-generated starters are never sent to a paid third-party API.

Wikipedia's REST API meets all five criteria and is the natural choice for an open-source, local-first tool.

---

### Wikipedia REST API — Technical Details

#### Endpoint Used

```
GET https://en.wikipedia.org/api/rest_v1/page/summary/{title}
```

**Response Fields Used:**

| Field | Usage |
|---|---|
| `title` | Confirmed entity name |
| `extract` | Plain-text summary for claim comparison |
| `timestamp` | Last edit date (freshness indicator) |
| `type` | `"standard"` vs `"disambiguation"` (filter ambiguous results) |

**Example Call:**
```python
import httpx

async def fetch_wikipedia_summary(entity: str) -> dict | None:
    url = f"https://en.wikipedia.org/api/rest_v1/page/summary/{entity.replace(' ', '_')}"
    async with httpx.AsyncClient(timeout=5.0) as client:
        response = await client.get(url)
        if response.status_code == 200:
            data = response.json()
            if data.get("type") != "disambiguation":
                return {
                    "title": data.get("title"),
                    "extract": data.get("extract", ""),
                    "timestamp": data.get("timestamp")
                }
    return None
```

---

### Fact-Check Pipeline

The fact-check pipeline operates in three stages after conversation starters are generated.

#### Stage 1 — Named Entity Recognition (NER)

Entities are extracted from each starter using a lightweight regex-based approach combined with spaCy's `en_core_web_sm` model:

```python
import spacy

nlp = spacy.load("en_core_web_sm")

def extract_entities(text: str) -> list[str]:
    """Extract ORG, PERSON, PRODUCT, EVENT entities."""
    doc = nlp(text)
    entities = [
        ent.text for ent in doc.ents
        if ent.label_ in {"ORG", "PERSON", "PRODUCT", "EVENT", "GPE"}
    ]
    return list(set(entities))
```

#### Stage 2 — Wikipedia Lookup

Each extracted entity is looked up via the Wikipedia REST API. Results are cached in-session to avoid redundant calls.

```python
async def check_entities(entities: list[str]) -> dict[str, dict]:
    results = {}
    for entity in entities:
        summary = await fetch_wikipedia_summary(entity)
        results[entity] = summary
    return results
```

#### Stage 3 — Confidence Annotation

Each starter receives a `fact_check_status` based on entity lookup outcomes:

```python
def annotate_starter(starter: str, entity_results: dict) -> str:
    entities = extract_entities(starter)
    found = [e for e in entities if entity_results.get(e) is not None]
    not_found = [e for e in entities if entity_results.get(e) is None]
    
    if not entities:
        return "unverified"       # No checkable claims → neutral
    elif len(not_found) == 0:
        return "verified"         # All entities confirmed on Wikipedia
    elif len(found) > 0:
        return "warning"          # Some entities confirmed, some not
    else:
        return "unverified"       # No entities found on Wikipedia
```

**Status Display in UI:**

| Status | Badge | Meaning |
|---|---|---|
| `verified` | ✅ Verified | All named entities confirmed on Wikipedia |
| `unverified` | ℹ️ Unverified | No checkable factual claims made |
| `warning` | ⚠️ Check Facts | One or more entities could not be confirmed |

---

### Fact-Check Limitations and Mitigations

| Limitation | Mitigation |
|---|---|
| Wikipedia has coverage gaps in niche domains | Status `unverified` is displayed as neutral, not incorrect |
| Wikipedia can itself contain errors | Users are reminded that fact-check is a confidence indicator, not absolute truth |
| NER may miss entities or extract false positives | spaCy's `en_core_web_sm` is augmented with regex patterns for common tech entities |
| API may be unavailable (network issues) | Timeout (5s) with graceful fallback to `unverified` status for all starters |

---

## 6. Local JSON Persistence — History and Feedback

### Design Decision — Why Local JSON

For an MVP that runs locally without requiring a database server, JSON files are the optimal persistence choice:

- **Zero infrastructure overhead** — no PostgreSQL, no SQLite setup required.
- **Human-readable** — developers can inspect and debug data directly.
- **Portable** — the entire application state travels with the project directory.
- **Sufficient for MVP scale** — a typical user will generate ≤ 500 sessions total.

JSON persistence is explicitly architected as **replaceable** — the `HistoryService` and `FeedbackService` classes abstract all read/write operations so that swapping to a database backend requires only changing the service implementation, not the API or UI.

---

### History Persistence — `data/history.json`

The history file is a JSON array of session objects. Each session represents one complete generation run.

**Schema:**
```json
[
  {
    "session_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "timestamp": "2026-07-07T07:00:00+05:30",
    "event_name": "KubeCon North America 2026",
    "user_role": "Senior Machine Learning Engineer",
    "user_interests": ["MLOps", "Kubernetes", "model deployment"],
    "extracted_themes": [
      {"theme": "cloud computing",       "confidence": 0.9312},
      {"theme": "machine learning",      "confidence": 0.8914},
      {"theme": "software engineering",  "confidence": 0.8201}
    ],
    "starters": [
      {
        "starter_id": "s-001",
        "text": "I've been exploring Kubernetes-native model serving...",
        "fact_check_status": "verified",
        "fact_check_entities": ["Kubernetes", "KServe"],
        "feedback": "like"
      },
      {
        "starter_id": "s-002",
        "text": "With so many teams moving toward GitOps...",
        "fact_check_status": "unverified",
        "fact_check_entities": [],
        "feedback": null
      }
    ]
  }
]
```

**Service Operations:**
```python
class HistoryService:
    def save_session(self, session: SessionData) -> str: ...
    def get_all_sessions(self) -> list[SessionData]: ...
    def get_session_by_id(self, session_id: str) -> SessionData | None: ...
    def update_starter_feedback(self, session_id: str, starter_id: str, feedback: str): ...
    def delete_session(self, session_id: str): ...
```

---

### Feedback Persistence — `data/feedback.json`

The feedback file logs all Like/Dislike interactions as a flat array for easy analytics processing.

**Schema:**
```json
[
  {
    "feedback_id": "fb-0001",
    "session_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "starter_id": "s-001",
    "starter_text": "I've been exploring Kubernetes-native model serving...",
    "sentiment": "like",
    "timestamp": "2026-07-07T07:15:00+05:30",
    "event_name": "KubeCon North America 2026",
    "user_role": "Senior Machine Learning Engineer"
  }
]
```

**Service Operations:**
```python
class FeedbackService:
    def record_feedback(self, feedback: FeedbackData) -> str: ...
    def get_all_feedback(self) -> list[FeedbackData]: ...
    def get_feedback_by_session(self, session_id: str) -> list[FeedbackData]: ...
    def get_feedback_summary(self) -> dict: ...
    # Returns: {"total": 42, "likes": 31, "dislikes": 11, "like_rate": 0.738}
```

---

## 7. Streamlit Dark-Themed UI

### Design Principles

The user interface is built with Streamlit using the following design principles:

1. **Dark theme** — reduces eye strain during event preparation (often done at night or in dim environments).
2. **Single-page feel with tabs** — users don't navigate to different pages; all functionality is accessible via tabs.
3. **Progressive disclosure** — advanced options (number of starters, confidence threshold) are hidden in expandable sections.
4. **Immediate feedback** — all actions produce visible responses within 1 second (or a spinner is shown for longer operations).
5. **Mobile-aware** — while not fully mobile-responsive, column layouts ensure reasonable rendering on screens ≥ 800px wide.

---

### Theme Configuration — `.streamlit/config.toml`

```toml
[theme]
base = "dark"
primaryColor = "#7C3AED"          # Purple accent
backgroundColor = "#0F0F0F"       # Near-black background
secondaryBackgroundColor = "#1A1A2E"  # Card/sidebar background
textColor = "#E2E8F0"             # Light gray text
font = "sans serif"
```

---

### Page Structure

#### Tab 1 — Generator

```
┌─────────────────────────────────────────────────────────┐
│  🤝 Personalized Networking Assistant                   │
├─────────────────────────────────────────────────────────┤
│  [Generate] [History] [Feedback]                        │
├─────────────────────────────────────────────────────────┤
│  📋 Event Details                                       │
│  ┌──────────────────────────────────┐                   │
│  │ Event Name: [__________________] │                   │
│  │ Your Role:  [__________________] │                   │
│  │ Interests:  [__________________] │                   │
│  └──────────────────────────────────┘                   │
│                                                         │
│  ⚙️ Advanced Options (collapsed)                        │
│  ┌──────────────────────────────────┐                   │
│  │ Number of Starters: [3] [slider] │                   │
│  └──────────────────────────────────┘                   │
│                                                         │
│  [ 🚀 Generate Conversation Starters ]                  │
│                                                         │
│  ─────── Generated Starters ───────                     │
│  ┌──────────────────────────────────┐                   │
│  │ ✅ Verified                      │                   │
│  │ "I've been exploring Kubernetes- │                   │
│  │  native model serving with       │                   │
│  │  KServe..."                      │                   │
│  │                    [👍] [👎] [📋] │                   │
│  └──────────────────────────────────┘                   │
│  ┌──────────────────────────────────┐                   │
│  │ ℹ️ Unverified                    │                   │
│  │ "With so many teams moving       │                   │
│  │  toward GitOps..."               │                   │
│  │                    [👍] [👎] [📋] │                   │
│  └──────────────────────────────────┘                   │
└─────────────────────────────────────────────────────────┘
```

#### Tab 2 — History

```
┌─────────────────────────────────────────────────────────┐
│  📚 Session History                                     │
│                                                         │
│  Filter: [All Events ▼]  Sort: [Newest First ▼]        │
│                                                         │
│  ┌──────────────────────────────────┐                   │
│  │ 📅 2026-07-07 07:00              │                   │
│  │ KubeCon North America 2026       │                   │
│  │ Role: Senior ML Engineer         │                   │
│  │ Themes: cloud computing, ML...   │                   │
│  │ Starters: 3 | Liked: 2 | 👎: 0  │                   │
│  │                   [View Details] │                   │
│  └──────────────────────────────────┘                   │
└─────────────────────────────────────────────────────────┘
```

#### Tab 3 — Feedback Analytics

```
┌─────────────────────────────────────────────────────────┐
│  📊 Feedback Summary                                    │
│                                                         │
│  Total Starters Generated:  42                          │
│  Liked:                     31  (73.8%)                 │
│  Disliked:                  11  (26.2%)                 │
│                                                         │
│  Most Used Themes:                                      │
│  ████████████  cloud computing  (12)                    │
│  ██████████    machine learning (9)                     │
│  ████████      cybersecurity    (7)                     │
└─────────────────────────────────────────────────────────┘
```

---

## 8. User Flow — End-to-End

The following describes the complete user journey from opening the application to receiving and rating personalized conversation starters.

```
USER ACTION                    SYSTEM ACTION
─────────────────────────────────────────────────────────────────────

1. Open app (streamlit run)
                               → Streamlit loads; FastAPI starts via
                                 subprocess or separate terminal
                               → BART and GPT-2 models load lazily
                                 (first request triggers load)

2. Navigate to Generator tab
                               → UI renders input form

3. Enter event details:
   - Event Name
   - Professional Role
   - Interests (comma-separated)
   - (Optional) Number of starters
                               → Form validation (all fields non-empty)

4. Click "Generate Starters"
                               → UI shows spinner: "Analyzing event..."
                               → POST /api/classify
                                 → BART classifies event → top 3 themes
                               → UI updates: "Generating starters..."
                               → POST /api/generate
                                 → Prompt built with user data + themes
                                 → GPT-2 generates raw text
                                 → Parser extracts starters
                               → UI updates: "Fact-checking..."
                               → POST /api/factcheck
                                 → NER extracts entities from starters
                                 → Wikipedia API queried per entity
                                 → Starters annotated with status
                               → POST /api/history/save
                                 → Session saved to history.json

5. Review starters in UI
                               → Each starter shown with:
                                 - Fact-check badge (✅/ℹ️/⚠️)
                                 - Full text
                                 - 👍 Like, 👎 Dislike, 📋 Copy buttons

6. Click 👍 or 👎 on a starter
                               → POST /api/feedback
                                 → Feedback recorded to feedback.json
                                 → History entry updated in history.json
                               → UI shows brief success toast

7. Click 📋 Copy
                               → Starter text copied to clipboard
                               → Toast: "Copied to clipboard!"

8. Navigate to History tab
                               → GET /api/history
                                 → All sessions loaded from history.json
                               → Sessions displayed newest-first
                               → User can expand any session to see all starters

9. Navigate to Feedback tab
                               → GET /api/feedback
                                 → Feedback data loaded and aggregated
                               → Summary metrics and charts displayed
```

---

## 9. System Boundaries and Interfaces

### What's Inside the System

```
INTERNAL (runs on user's machine):
  ✅ Streamlit frontend
  ✅ FastAPI backend
  ✅ BART model (downloaded to ~/.cache/huggingface)
  ✅ GPT-2 model (downloaded to ~/.cache/huggingface)
  ✅ spaCy NER model (en_core_web_sm)
  ✅ history.json, feedback.json
  ✅ All service layer logic
```

### What's Outside the System

```
EXTERNAL (network calls):
  🌐 Wikipedia REST API — read-only, no user data sent
                          only entity names are sent as URL parameters

NOT USED (explicitly excluded):
  ❌ OpenAI API
  ❌ Google Cloud / AWS / Azure APIs
  ❌ Any analytics or telemetry service
  ❌ Any authentication or user management service
```

### Interface Contracts

#### Streamlit → FastAPI

All calls from Streamlit to FastAPI are made via `httpx` (async HTTP client) to `http://localhost:8000/api/*`. The FastAPI server must be running before Streamlit calls are made.

```python
# In streamlit app
import httpx

BASE_URL = os.getenv("API_BASE_URL", "http://localhost:8000")

async def call_api(endpoint: str, payload: dict) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.post(f"{BASE_URL}/api/{endpoint}", json=payload)
        response.raise_for_status()
        return response.json()
```

#### FastAPI → Services

Services are called as pure Python functions (synchronous for model inference, async for I/O):

```python
# In FastAPI route handler
@router.post("/classify")
async def classify_endpoint(request: ClassifyRequest):
    themes = classify_event_themes(request.event_name)  # sync, CPU-bound
    return ClassifyResponse(themes=themes)
```

---

## 10. Ethical AI Considerations

### Transparency

- Every AI-generated starter is clearly labeled as **AI-generated** in the UI.
- The fact-check system provides **confidence annotations** rather than false certainty.
- Users are informed in the UI that GPT-2 has a **2019 knowledge cutoff** and may produce outdated references.

### Data Privacy

- **No user data is sent to any external AI API.** All model inference runs locally.
- The only external call is to Wikipedia's public REST API, which sends only entity names as URL parameters — no personal information.
- History and feedback are stored only on the user's local machine in JSON files under the project's `data/` directory.

### Bias Awareness

- GPT-2 was trained on internet text and may reflect biases present in that corpus. Generated starters are reviewed by the user before use.
- The candidate label set for BART is reviewed to ensure no professional domain is systematically under-represented.
- The team commits to expanding and diversifying the candidate label set in future versions.

### Intellectual Honesty

- The Wikipedia fact-check is explicitly described as a **confidence indicator**, not a truth oracle. Users are encouraged to verify important claims independently.
- The feedback system is disclosed as a **local data store** only — no feedback data is sent to the development team or any third party.

### Limitations Disclosed to Users

The UI includes a visible disclaimer:
> *"Starters are AI-generated and may require review. The fact-check feature uses Wikipedia as a reference and may not catch all inaccuracies. Use your judgment before using any starter in conversation."*

---

## 11. Expected Outcomes and Impact

### For Individual Users

| Outcome | Measure |
|---|---|
| Reduced pre-event anxiety | Self-reported confidence score increase (pre/post app use survey) |
| Faster preparation time | Target: < 5 minutes from app open to starters ready |
| Higher quality conversations | Self-reported satisfaction with networking conversations |
| Growing personal toolkit | Number of liked starters saved in history over time |

### For the Field of Networking Tools

The Personalized Networking Assistant demonstrates three architectural advances over existing networking tools:

1. **Theme-conditioned generation** — combining a classifier + generator in a local pipeline (no cloud required).
2. **Factual grounding via open APIs** — showing that fact-checking can be added to generation pipelines without paid APIs.
3. **Feedback-informed curation** — capturing per-item feedback at the interaction level rather than session level.

### Quantified Impact Targets (Post-MVP)

| KPI | Baseline (current tools) | Target (after 3 months of use) |
|---|---|---|
| Starters rated "useful" by users | 40% (industry estimate) | ≥ 70% |
| Pre-event preparation time | 20–30 minutes | ≤ 5 minutes |
| User task completion rate | N/A | 100% (Scenario 3) |
| BART theme accuracy (top-1) | N/A | ≥ 80% |
| SUS usability score | N/A | ≥ 68 |

### Roadmap Beyond MVP

The architecture is designed to support the following future enhancements without major refactoring:

1. **Fine-tuned GPT-2** — trained on a curated dataset of high-rated networking starters.
2. **Feedback-adaptive prompting** — dynamically adjusting prompt style based on accumulated like/dislike patterns.
3. **User profiles** — multi-user support with per-user history and feedback namespacing.
4. **Cloud deployment** — FastAPI + Streamlit deployable to Render, Railway, or Hugging Face Spaces with minimal configuration changes.
5. **Additional fact sources** — augmenting Wikipedia with ArXiv API (for academic events) and Crunchbase (for startup events).

---

*Document prepared by the Personalized Networking Assistant Project Team.*  
*Team Lead: Shaik Sumiya Zainab | Members: Naga Jagan Mohan Rao Thattukolla, Satvika Tallam, Tejesh Velivela*
