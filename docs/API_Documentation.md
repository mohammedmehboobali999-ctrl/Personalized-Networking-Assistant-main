# Exhaustive API Reference and Architecture Documentation
**Project:** Personalized Networking Assistant  
**Repository:** `networking-assistant`  
**Document Version:** 1.0.0  
**Date:** July 2026  
**Authors:** mehboob ali mohammed  

---

## 1. Executive Summary and Architecture Overview

The **Personalized Networking Assistant** exposes a high-performance RESTful Application Programming Interface (API) built using **FastAPI** on port `8000`. The API serves as the core orchestration gateway, bridging asynchronous HTTP requests from the Streamlit frontend (`port 8501`) with local CPU-bound HuggingFace NLP models (`facebook/bart-large-mnli` and `gpt2` 124M), external Wikipedia fact-checking services, and local atomic JSON persistence.

This document provides exhaustive reference documentation for all seven API endpoints implemented in `app/routes/api.py`. For every endpoint, this specification defines HTTP methods, routing paths, architectural roles, request/response JSON schemas validated via Pydantic v2, error handling structures, ready-to-execute `curl` commands, and realistic JSON payload examples.

---

## 2. General API Conventions & System Architecture

### 2.1 Base URL & Network Binding
- **Local Development / Direct Uvicorn:** `http://localhost:8000`
- **Containerized Docker Deployment:** `http://api:8000` (Internal Docker bridge network)
- **Interactive Swagger Documentation:** `http://localhost:8000/docs`
- **ReDoc API Specification:** `http://localhost:8000/redoc`

### 2.2 Content-Type & Data Serialization
All POST and PUT requests must specify the `Content-Type: application/json` header. Request bodies and response payloads are serialized using Pydantic v2 strict models (`app/models/schemas.py`), ensuring robust type checking, automatic data coercion, and descriptive validation errors.

### 2.3 Authentication & Stateless Design
In accordance with Minimum Viable Product (MVP) architectural constraints and local data privacy requirements, the API currently operates without authentication tokens or session cookies. All requests are stateless, executing locally on the user's workstation. *(Note: Future enhancements outline OAuth2 and JWT bearer token integration for multi-user enterprise deployments).*

### 2.4 Race-Condition-Free Atomic File Persistence
To eliminate cloud database dependencies while ensuring complete data integrity, all historical conversation sessions and user feedback ratings are persisted locally to `data/history.json` and `data/feedback.json`. To prevent JSON corruption or file lock collisions during concurrent asynchronous POST requests, backend endpoints implement **atomic file persistence** via Python `pathlib.Path`:
1. Data is written to a temporary file (`history.json.tmp.<uuid>`).
2. Once the file buffer is flushed and closed, the temporary file atomically renames and overwrites the target file using POSIX/Windows filesystem atomic rename operations.

### 2.5 Standardized Error Handling Architecture
When an API request fails, FastAPI returns a structured JSON error payload conforming to RFC 7807 problem details.

#### HTTP 422 Unprocessable Entity (Pydantic Validation Error Schema):
```json
{
  "detail": [
    {
      "type": "string_too_short",
      "loc": ["body", "event_name"],
      "msg": "String should have at least 3 characters",
      "input": "AI",
      "ctx": {"min_length": 3}
    }
  ]
}
```

#### HTTP 400 / 500 / 503 Server & Service Error Schema:
```json
{
  "detail": "External knowledge base currently unreachable. Wikipedia REST API connection timed out after 5.0 seconds."
}
```

---

## 3. Exhaustive Endpoint Reference

### 3.1 System Health & Metadata Check
- **Endpoint Path:** `/`
- **HTTP Method:** `GET`
- **Architectural Role:** Serves as the primary liveness and readiness probe for Docker container health checks and frontend liveness verification. Confirms that the FastAPI ASGI server is running and returns core application metadata.

#### Query Parameters / Path Parameters
*None.*

#### Request Body Schema
*None (GET Request).*

#### Response Schema (200 OK)
| Field Name | Data Type | Description | Validation Rule / Constraint |
| :--- | :--- | :--- | :--- |
| `status` | String | System operational status. | Must be exactly `"online"`. |
| `app_name` | String | Configured application name from `app/config.py`. | Non-empty string. |
| `version` | String | Current Semantic Versioning tag of the repository. | Pattern: `^\d+\.\d+\.\d+$`. |
| `timestamp` | String (ISO-8601) | Current UTC server timestamp when request was handled. | Valid ISO-8601 UTC date-time string. |

#### Error Responses
- **500 Internal Server Error:** Returned if internal server configuration or environment variables fail to load.

#### Complete `curl` Example Request
```bash
curl -X GET "http://localhost:8000/" \
     -H "Accept: application/json"
```

#### Complete JSON Example Response
```json
{
  "status": "online",
  "app_name": "Personalized Networking Assistant",
  "version": "1.0.0",
  "timestamp": "2026-07-07T17:15:00.123456Z"
}
```

---

### 3.2 Zero-Shot Event Theme Extraction
- **Endpoint Path:** `/api/v1/analyze-event`
- **HTTP Method:** `POST`
- **Architectural Role:** Analyzes raw networking event descriptions or conference names and classifies them into professional networking themes using the zero-shot classification pipeline powered by HuggingFace `facebook/bart-large-mnli` (`app/services/ai_service.py`).

#### Query Parameters / Path Parameters
*None.*

#### Request Body Schema (`EventAnalysisRequest`)
| Field Name | Data Type | Required | Description | Validation Rules & Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `event_name` | String | **Yes** | Title or name of the networking event, conference, or meetup. | Minimum length: 3 chars.<br>Maximum length: 150 chars. |
| `event_description` | String | **Yes** | Detailed background description, agenda topics, or target audience of the event. | Minimum length: 10 chars.<br>Maximum length: 2000 chars. |
| `candidate_themes` | List[String] | No | Optional custom list of themes to classify against. Defaults to standard professional domains if omitted. | Array length: 2–10 items.<br>Each string: 2–50 chars. |

#### Response Schema (200 OK - `EventAnalysisResponse`)
| Field Name | Data Type | Description | Validation Rule / Constraint |
| :--- | :--- | :--- | :--- |
| `primary_theme` | String | The highest-probability professional theme extracted by BART-large-mnli. | Non-empty string matching candidate list. |
| `confidence_score` | Float | Normalized probability score assigned to the primary theme. | Value range: `0.0` to `1.0`. |
| `all_scores` | Dict[String, Float] | Complete mapping of all candidate themes and their respective classification probabilities. | Key: Theme name.<br>Value: Probability float (summing to approx 1.0). |
| `processing_time_ms` | Float | Time elapsed in milliseconds during CPU model inference. | Positive float value. |

#### Error Responses
- **422 Unprocessable Entity:** Returned if `event_name` is under 3 characters or `event_description` is missing.
- **500 Internal Server Error:** Returned if the HuggingFace BART model fails to initialize from `./models_cache` or encounters a PyTorch CPU runtime error.

#### Complete `curl` Example Request
```bash
curl -X POST "http://localhost:8000/api/v1/analyze-event" \
     -H "Content-Type: application/json" \
     -H "Accept: application/json" \
     -d '{
       "event_name": "Global AI & Machine Learning Symposium 2026",
       "event_description": "An annual gathering of artificial intelligence researchers, data scientists, and ML engineers discussing large language models, neural network architectures, and AI ethics.",
       "candidate_themes": [
         "Artificial Intelligence & ML",
         "Cloud Computing & DevOps",
         "Venture Capital & Startups",
         "Biotechnology & Healthcare"
       ]
     }'
```

#### Complete JSON Example Response
```json
{
  "primary_theme": "Artificial Intelligence & ML",
  "confidence_score": 0.8942,
  "all_scores": {
    "Artificial Intelligence & ML": 0.8942,
    "Cloud Computing & DevOps": 0.0615,
    "Venture Capital & Startups": 0.0311,
    "Biotechnology & Healthcare": 0.0132
  },
  "processing_time_ms": 3420.5
}
```

---

### 3.3 Conversation Starter Generation
- **Endpoint Path:** `/api/v1/generate-conversation`
- **HTTP Method:** `POST`
- **Architectural Role:** The primary generative engine of the platform. Synthesizes event context, user role, and target attendee profiles to generate 3 to 5 tailored conversation starters using local CPU inference via HuggingFace `gpt2` (124M parameters). Automatically records the generated session to `data/history.json` using atomic file persistence.

#### Query Parameters / Path Parameters
*None.*

#### Request Body Schema (`ConversationGenerationRequest`)
| Field Name | Data Type | Required | Description | Validation Rules & Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `event_name` | String | **Yes** | Title or name of the professional networking engagement. | Minimum length: 3 chars.<br>Maximum length: 150 chars. |
| `user_role` | String | **Yes** | The professional title or role of the user generating the starters (e.g., *"Junior AI Engineer"*). | Minimum length: 2 chars.<br>Maximum length: 100 chars. |
| `target_role` | String | **Yes** | The role or profile of the individual the user wants to initiate a conversation with (e.g., *"Senior Data Scientist"*, *"Keynote Speaker"*). | Minimum length: 2 chars.<br>Maximum length: 100 chars. |
| `industry_domain` | String | **Yes** | Industry sector (e.g., *"Fintech"*, *"Generative AI"*, *"Healthcare"*). | Minimum length: 2 chars.<br>Maximum length: 80 chars. |
| `extracted_theme` | String | No | Optional pre-extracted theme from `/api/v1/analyze-event`. If omitted, backend infers theme automatically. | Maximum length: 100 chars. |
| `starter_count` | Integer | No | Desired number of conversation starters to generate. | Minimum value: `1`<br>Maximum value: `5`<br>Default value: `3` |

#### Response Schema (200 OK - `ConversationGenerationResponse`)
| Field Name | Data Type | Description | Validation Rule / Constraint |
| :--- | :--- | :--- | :--- |
| `session_id` | String (UUIDv4) | Unique identifier assigned to this generation session and stored in `data/history.json`. | Valid UUIDv4 format (`8-4-4-4-12` hex chars). |
| `starters` | List[String] | Array of generated conversation starters tailored to the input profile. | Array length matches `starter_count`. Each string: 30–300 chars. |
| `context_summary` | String | Brief synthesis of the prompt context provided to the GPT-2 inference engine. | Non-empty string. |
| `created_at` | String (ISO-8601) | UTC timestamp when the starters were generated and atomically logged. | Valid ISO-8601 UTC date-time string. |
| `model_used` | String | Name of the generative AI model utilized for inference. | Must be `"gpt2-124m-cpu"`. |

#### Error Responses
- **422 Unprocessable Entity:** Returned if `starter_count` exceeds 5 or required fields like `user_role` are empty.
- **500 Internal Server Error:** Returned if CPU memory exhaustion occurs during GPT-2 text generation or if atomic file writing to `data/history.json` fails due to filesystem permissions.

#### Complete `curl` Example Request
```bash
curl -X POST "http://localhost:8000/api/v1/generate-conversation" \
     -H "Content-Type: application/json" \
     -H "Accept: application/json" \
     -d '{
       "event_name": "NeurIPS Conference 2026 Networking Mixer",
       "user_role": "Machine Learning Graduate Student",
       "target_role": "Senior AI Research Scientist",
       "industry_domain": "Deep Learning & Generative AI",
       "extracted_theme": "Artificial Intelligence & ML",
       "starter_count": 3
     }'
```

#### Complete JSON Example Response
```json
{
  "session_id": "c3a9f18e-4d2b-48c1-8f5a-9b2e4c7d6a10",
  "starters": [
    "I really enjoyed the recent discussion on scaling generative AI architectures; as an ML graduate student, I'm curious about your perspective on handling inference bottlenecks in small language models.",
    "Hello! I saw your recent work in Deep Learning at NeurIPS. How do you see zero-shot classification evolving for real-time edge applications over the next year?",
    "Hi there! I'm researching parameter-efficient fine-tuning methods. Given your experience as a Senior AI Research Scientist, what challenges do you think are most critical when deploying these models to production?"
  ],
  "context_summary": "ML Graduate Student initiating dialogue with Senior AI Research Scientist at NeurIPS Conference 2026 within Deep Learning domain.",
  "created_at": "2026-07-07T17:18:22.451092Z",
  "model_used": "gpt2-124m-cpu"
}
```

---

### 3.4 Real-Time Wikipedia Fact Verification
- **Endpoint Path:** `/api/v1/fact-check`
- **HTTP Method:** `GET`
- **Architectural Role:** Cross-references technical acronyms, industry claims, or organization names against the external Wikipedia REST API (`https://en.wikipedia.org/api/rest_v1/page/summary/{query}`). Implements automated confidence scoring and defensive fallback mechanisms to prevent AI hallucinations during professional networking discussions.

#### Query Parameters / Path Parameters
| Parameter Name | Type | Required | Description | Validation Rules & Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `query` | String (Query Param) | **Yes** | The technical claim, concept, or entity name to verify against Wikipedia. | Minimum length: 2 chars.<br>Maximum length: 100 chars.<br>URL-encoded string. |

#### Request Body Schema
*None (GET Request).*

#### Response Schema (200 OK - `FactCheckResponse`)
| Field Name | Data Type | Description | Validation Rule / Constraint |
| :--- | :--- | :--- | :--- |
| `query` | String | The normalized search query submitted by the user. | Matches input parameter. |
| `status` | String | Verification status of the entity against Wikipedia. | Enum: `"verified"`, `"unverified"`, `"disambiguation"`, `"error"`. |
| `confidence` | String | Calculated confidence level based on Wikipedia page exactness and summary availability. | Enum: `"high"`, `"medium"`, `"low"`. |
| `summary` | String | Extracted introductory summary text from Wikipedia, or a structured fallback message if unverified. | Maximum length: 1000 chars. |
| `source_url` | String (URL / null) | Direct canonical URL to the Wikipedia article, or `null` if unverified. | Valid HTTP/HTTPS URL or null. |
| `checked_at` | String (ISO-8601) | UTC timestamp when the fact-check query was executed. | Valid ISO-8601 UTC date-time string. |

#### Error Responses
- **400 Bad Request:** Returned if `query` parameter is missing or consists entirely of whitespace.
- **422 Unprocessable Entity:** Returned if query string exceeds 100 characters.
- *(Note: External Wikipedia HTTP 404 Not Found or 503 Service Unavailable errors are caught defensively by backend exception handlers and returned as a clean `200 OK` response with `"status": "unverified"` and `"confidence": "low"`).*

#### Complete `curl` Example Request
```bash
curl -X GET "http://localhost:8000/api/v1/fact-check?query=Transformer+(machine+learning+model)" \
     -H "Accept: application/json"
```

#### Complete JSON Example Response
```json
{
  "query": "Transformer (machine learning model)",
  "status": "verified",
  "confidence": "high",
  "summary": "A transformer is a deep learning architecture developed by Google and based on the multi-head attention mechanism, proposed in a 2017 paper 'Attention Is All You Need'. It has become the foundational model architecture for modern large language models.",
  "source_url": "https://en.wikipedia.org/wiki/Transformer_(deep_learning_architecture)",
  "checked_at": "2026-07-07T17:20:11.890123Z"
}
```

---

### 3.5 Historical Generation Retrieval
- **Endpoint Path:** `/api/v1/history`
- **HTTP Method:** `GET`
- **Architectural Role:** Retrieves historical conversation starter generation sessions stored in local atomic JSON persistence (`data/history.json`). Supports optional pagination and keyword searching to allow users to review past networking preparations in the Streamlit UI.

#### Query Parameters / Path Parameters
| Parameter Name | Type | Required | Description | Validation Rules & Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `limit` | Integer (Query) | No | Maximum number of historical sessions to return in a single request. | Minimum value: `1`<br>Maximum value: `100`<br>Default value: `20` |
| `offset` | Integer (Query) | No | Number of historical records to skip for pagination. | Minimum value: `0`<br>Default value: `0` |
| `search` | String (Query) | No | Optional keyword filter to search across event names, industry domains, and generated starters. | Maximum length: 50 chars. |

#### Request Body Schema
*None (GET Request).*

#### Response Schema (200 OK - `HistoryListResponse`)
| Field Name | Data Type | Description | Validation Rule / Constraint |
| :--- | :--- | :--- | :--- |
| `total_count` | Integer | Total number of historical sessions stored in `data/history.json` matching search criteria. | Non-negative integer (`>= 0`). |
| `limit` | Integer | Applied limit parameter for pagination. | Matches request parameter. |
| `offset` | Integer | Applied offset parameter for pagination. | Matches request parameter. |
| `sessions` | List[Dict] | Array of historical generation session objects. | Array length `<= limit`. |

**Structure of Each Object in `sessions` List:**
```json
{
  "session_id": "c3a9f18e-4d2b-48c1-8f5a-9b2e4c7d6a10",
  "event_name": "NeurIPS Conference 2026 Networking Mixer",
  "user_role": "Machine Learning Graduate Student",
  "target_role": "Senior AI Research Scientist",
  "industry_domain": "Deep Learning & Generative AI",
  "starters": ["I really enjoyed the recent discussion..."],
  "created_at": "2026-07-07T17:18:22.451092Z"
}
```

#### Error Responses
- **422 Unprocessable Entity:** Returned if `limit` is negative or exceeds 100.
- **500 Internal Server Error:** Returned if `data/history.json` is corrupted or unreadable due to filesystem permission errors.

#### Complete `curl` Example Request
```bash
curl -X GET "http://localhost:8000/api/v1/history?limit=5&offset=0&search=NeurIPS" \
     -H "Accept: application/json"
```

#### Complete JSON Example Response
```json
{
  "total_count": 1,
  "limit": 5,
  "offset": 0,
  "sessions": [
    {
      "session_id": "c3a9f18e-4d2b-48c1-8f5a-9b2e4c7d6a10",
      "event_name": "NeurIPS Conference 2026 Networking Mixer",
      "user_role": "Machine Learning Graduate Student",
      "target_role": "Senior AI Research Scientist",
      "industry_domain": "Deep Learning & Generative AI",
      "starters": [
        "I really enjoyed the recent discussion on scaling generative AI architectures; as an ML graduate student, I'm curious about your perspective on handling inference bottlenecks in small language models.",
        "Hello! I saw your recent work in Deep Learning at NeurIPS. How do you see zero-shot classification evolving for real-time edge applications over the next year?"
      ],
      "created_at": "2026-07-07T17:18:22.451092Z"
    }
  ]
}
```

---

### 3.6 User Feedback Submission
- **Endpoint Path:** `/api/v1/feedback`
- **HTTP Method:** `POST`
- **Architectural Role:** Captures empirical user evaluations on generated conversation starters. Records binary thumbs-up/down actions and 5-star rating scores, writing them atomically to `data/feedback.json` to enable continuous prompt engineering evaluation.

#### Query Parameters / Path Parameters
*None.*

#### Request Body Schema (`FeedbackSubmissionRequest`)
| Field Name | Data Type | Required | Description | Validation Rules & Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `session_id` | String (UUIDv4) | **Yes** | The UUIDv4 identifier of the historical generation session being evaluated. | Must match an existing ID or valid UUIDv4 string (`8-4-4-4-12`). |
| `starter_index` | Integer | **Yes** | The 0-indexed position of the specific conversation starter being rated within the session. | Minimum value: `0`<br>Maximum value: `4`. |
| `rating_stars` | Integer | **Yes** | Quantitative star rating assigned by the user. | Minimum value: `1`<br>Maximum value: `5`. |
| `thumbs_action` | String | **Yes** | Binary sentiment evaluation button clicked in UI. | Must be exactly `"up"` or `"down"`. |
| `user_comments` | String | No | Optional qualitative feedback or notes explaining the rating. | Maximum length: 500 chars. |

#### Response Schema (200 OK - `FeedbackSubmissionResponse`)
| Field Name | Data Type | Description | Validation Rule / Constraint |
| :--- | :--- | :--- | :--- |
| `feedback_id` | String (UUIDv4) | Unique system identifier generated for this specific feedback record. | Valid UUIDv4 format (`8-4-4-4-12`). |
| `session_id` | String (UUIDv4) | Reference back to the evaluated session ID. | Matches request parameter. |
| `status` | String | Confirmation status of atomic persistence. | Must be exactly `"recorded"`. |
| `recorded_at` | String (ISO-8601) | UTC timestamp when feedback was written to `data/feedback.json`. | Valid ISO-8601 UTC date-time string. |

#### Error Responses
- **422 Unprocessable Entity:** Returned if `rating_stars` is outside the range `1–5` or `thumbs_action` is not `"up"` / `"down"`.
- **500 Internal Server Error:** Returned if atomic file locking fails during concurrent write operations to `data/feedback.json`.

#### Complete `curl` Example Request
```bash
curl -X POST "http://localhost:8000/api/v1/feedback" \
     -H "Content-Type: application/json" \
     -H "Accept: application/json" \
     -d '{
       "session_id": "c3a9f18e-4d2b-48c1-8f5a-9b2e4c7d6a10",
       "starter_index": 0,
       "rating_stars": 5,
       "thumbs_action": "up",
       "user_comments": "Highly relevant icebreaker! Mentioned scaling architectures which matched the keynote perfectly."
     }'
```

#### Complete JSON Example Response
```json
{
  "feedback_id": "f81d4fae-7dec-11d0-a765-00a0c91e6bf6",
  "session_id": "c3a9f18e-4d2b-48c1-8f5a-9b2e4c7d6a10",
  "status": "recorded",
  "recorded_at": "2026-07-07T17:23:45.678901Z"
}
```

---

### 3.7 Feedback Analytics & Aggregation Retrieval
- **Endpoint Path:** `/api/v1/feedback`
- **HTTP Method:** `GET`
- **Architectural Role:** Retrieves aggregated analytics and historical rating distributions from `data/feedback.json`. Utilized by the Streamlit Analytics Dashboard (`dashboard.py`) to render quantitative bar charts and KPI metrics.

#### Query Parameters / Path Parameters
| Parameter Name | Type | Required | Description | Validation Rules & Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `session_id` | String (Query) | No | Optional filter to retrieve feedback records exclusively for a specific session ID. | Valid UUIDv4 string if provided. |

#### Request Body Schema
*None (GET Request).*

#### Response Schema (200 OK - `FeedbackAnalyticsResponse`)
| Field Name | Data Type | Description | Validation Rule / Constraint |
| :--- | :--- | :--- | :--- |
| `total_feedback_count` | Integer | Total number of feedback ratings logged across all sessions. | Non-negative integer (`>= 0`). |
| `average_star_rating` | Float | Mean star rating across all recorded feedback entries. | Value range: `0.0` to `5.0`. |
| `thumbs_up_count` | Integer | Total count of positive (`"up"`) sentiment evaluations. | Non-negative integer (`<= total_feedback_count`). |
| `thumbs_down_count` | Integer | Total count of negative (`"down"`) sentiment evaluations. | Non-negative integer (`<= total_feedback_count`). |
| `rating_distribution` | Dict[String, Integer] | Mapping of star ratings (`"1"` through `"5"`) to their respective frequency counts. | Keys: `"1"`, `"2"`, `"3"`, `"4"`, `"5"`.<br>Values: Non-negative integers. |
| `recent_feedback` | List[Dict] | Array of the 10 most recent feedback submission objects. | Array length `<= 10`. |

**Structure of Each Object in `recent_feedback` List:**
```json
{
  "feedback_id": "f81d4fae-7dec-11d0-a765-00a0c91e6bf6",
  "session_id": "c3a9f18e-4d2b-48c1-8f5a-9b2e4c7d6a10",
  "starter_index": 0,
  "rating_stars": 5,
  "thumbs_action": "up",
  "user_comments": "Highly relevant icebreaker!",
  "recorded_at": "2026-07-07T17:23:45.678901Z"
}
```

#### Error Responses
- **422 Unprocessable Entity:** Returned if an invalid UUID format is supplied in the optional `session_id` filter parameter.
- **500 Internal Server Error:** Returned if `data/feedback.json` is missing or corrupted.

#### Complete `curl` Example Request
```bash
curl -X GET "http://localhost:8000/api/v1/feedback" \
     -H "Accept: application/json"
```

#### Complete JSON Example Response
```json
{
  "total_feedback_count": 42,
  "average_star_rating": 4.57,
  "thumbs_up_count": 38,
  "thumbs_down_count": 4,
  "rating_distribution": {
    "1": 0,
    "2": 1,
    "3": 3,
    "4": 9,
    "5": 29
  },
  "recent_feedback": [
    {
      "feedback_id": "f81d4fae-7dec-11d0-a765-00a0c91e6bf6",
      "session_id": "c3a9f18e-4d2b-48c1-8f5a-9b2e4c7d6a10",
      "starter_index": 0,
      "rating_stars": 5,
      "thumbs_action": "up",
      "user_comments": "Highly relevant icebreaker! Mentioned scaling architectures which matched the keynote perfectly.",
      "recorded_at": "2026-07-07T17:23:45.678901Z"
    }
  ]
}
```

---

## 4. Summary Table of API Endpoints

| HTTP Method | Routing Path | Primary Functional Description | Request Schema | Response Schema |
| :--- | :--- | :--- | :--- | :--- |
| **`GET`** | `/` | System health check and basic application metadata probe. | *None* | System status (`online`), version, timestamp |
| **`POST`** | `/api/v1/analyze-event` | Zero-shot professional theme extraction via HuggingFace BART-large-mnli. | `EventAnalysisRequest` | `EventAnalysisResponse` |
| **`POST`** | `/api/v1/generate-conversation` | Conversation starter generation via CPU-bound GPT-2 (124M) with atomic logging. | `ConversationGenerationRequest` | `ConversationGenerationResponse` |
| **`GET`** | `/api/v1/fact-check` | Real-time Wikipedia REST API fact verification with confidence scoring. | Query: `query` | `FactCheckResponse` |
| **`GET`** | `/api/v1/history` | Paginated and searchable retrieval of past networking generation sessions. | Query: `limit`, `offset`, `search` | `HistoryListResponse` |
| **`POST`** | `/api/v1/feedback` | Submission of user star ratings and thumbs-up/down empirical feedback. | `FeedbackSubmissionRequest` | `FeedbackSubmissionResponse` |
| **`GET`** | `/api/v1/feedback` | Retrieval of aggregated feedback analytics and rating distributions for UI dashboards. | Query: `session_id` *(optional)* | `FeedbackAnalyticsResponse` |
