# Brainstorming & Idea Prioritization
### Personalized Networking Assistant — AI-Powered Conversation Starter Web App

---

## 📋 Document Information

| Field | Details |
|---|---|
| **Project Name** | Personalized Networking Assistant |
| **Document Type** | Brainstorming & Idea Prioritization |
| **Developer** | mehboob ali mohammed |
| **Team Members** | mehboob ali mohammed |
| **Date** | July 2026 |
| **Version** | 1.0 |

---

## 1. Problem Identification

### 1.1 Context
Professionals attending networking events — conferences, career fairs, alumni meetups, and industry summits — consistently report that the most difficult moment is the very first one: **initiating a conversation with a stranger**. Despite knowing they need to network, most individuals freeze at the opening line, defaulting to generic phrases like *"So, what do you do?"* — which often fail to leave a memorable impression.

This problem is especially acute for:
- **Graduate students** transitioning into professional environments
- **Early-career professionals** attending their first industry conferences
- **Introverts and non-native English speakers** who feel additional cognitive load during spontaneous social interactions

### 1.2 Initial Problem Statement (Raw)
> *"People are bad at networking not because they don't want to network, but because they don't know how to start a conversation that feels personalized, natural, and relevant."*

### 1.3 Core Friction Points Identified
1. Generic openers feel awkward and transactional
2. Lack of real-time knowledge about the event topic or attendee background
3. Fear of saying something irrelevant or embarrassing
4. No structured tool to help prepare or adapt in real-time
5. Existing tools (LinkedIn, business card apps) don't generate *conversation content*

---

## 2. Brainstorming Sessions

> **Format:** Each session was time-boxed to 25 minutes, using a digital whiteboard (Miro). Ideas were captured raw — no filtering during sessions. Evaluation came after.

---

### 2.1 Session 1 — Team Lead + All Members
**Facilitator:** mehboob ali mohammed
**Participants:** Full team
**Date:** Week 1 of project kickoff
**Method:** Classic Brainstorm (no-judgment, free-form)

| # | Idea | Contributor |
|---|---|---|
| 1 | Build a chatbot that asks you about the event and the person you want to meet, then generates 5 tailored conversation starters | Sumiya |
| 2 | A template library: user picks a category (e.g., "tech startup founder") and gets pre-written scripts | mehboob ali mohammed |
| 3 | NLP classifier to detect the theme of an event from its description, then match tone of conversation starters | mehboob ali mohammed |
| 4 | Integrate Wikipedia API to pull real facts about the event topic and embed them in conversation starters | Satvika |
| 5 | Mobile app with push notification reminders before events with suggested openers | Tejesh |
| 6 | Use sentiment analysis to adjust the "warmth" of conversation starters (formal vs. casual) | Sumiya |
| 7 | Browser extension that scans a LinkedIn profile and generates starters based on it | Naga Jagan |
| 8 | GPT-based text generation that writes conversation starters from user-provided keywords | Satvika |
| 9 | A feedback loop where users rate the conversation starters and the system learns over time | Tejesh |
| 10 | Real-time event feed integration (Eventbrite/Meetup API) to pull context about attendees | Naga Jagan |
| 11 | Gamified system: "Networking XP" for every starter used and rated positively | Sumiya |

---

### 2.2 Session 2 — Technical Feasibility Deep-Dive
**Facilitator:** Naga Jagan Mohan Rao Thattukolla
**Participants:** Sumiya, Naga Jagan, Tejesh
**Date:** Week 1, Day 3
**Method:** SCAMPER (Substitute, Combine, Adapt, Modify, Put to other use, Eliminate, Reverse)

| SCAMPER Lens | Idea Generated |
|---|---|
| **Substitute** | Instead of GPT-4 (expensive), substitute with open-source GPT-2 for generation |
| **Combine** | Combine a zero-shot text classifier (BART) with a text generator (GPT-2) in a pipeline |
| **Adapt** | Adapt Wikipedia's open API to fact-check or enrich generated starters with real knowledge |
| **Modify** | Modify the output to allow toggling between formal/informal tone |
| **Put to other use** | Use BART's zero-shot classification, normally used for topic detection, to classify *event themes* |
| **Eliminate** | Eliminate the need for user login / profile creation to reduce friction |
| **Reverse** | Instead of asking "what should I say?", reverse it: "what topic would *they* enjoy talking about?" |

Additional ideas from this session:
- **Idea 12:** Use BART (`facebook/bart-large-mnli`) for zero-shot classification of event type
- **Idea 13:** Streamlit frontend for rapid prototyping without needing a custom React app
- **Idea 14:** FastAPI backend to serve model inference endpoints cleanly
- **Idea 15:** Wikipedia fact injection pipeline — search for event topic, extract snippet, inject into prompt

---

### 2.3 Session 3 — User-Centric Brainstorm
**Facilitator:** Satvika Tallam
**Participants:** Sumiya, Satvika
**Date:** Week 2, Day 1
**Method:** "How Might We" (HMW) Questions

| HMW Question | Spawned Idea |
|---|---|
| HMW make it feel *personal* without needing deep user data? | Let user describe the person they want to meet in 1–2 sentences |
| HMW ensure the content is accurate and not hallucinated? | Fact-check final outputs using Wikipedia API |
| HMW allow non-technical users to use this easily? | Streamlit UI with clear input labels, sliders, and buttons |
| HMW make it work for any event type? | Zero-shot BART classification so no pre-training needed for new event themes |
| HMW handle diverse communication styles? | Add a tone selector: Professional, Friendly, Curious, Humorous |
| HMW allow quick feedback without a full rating system? | Thumbs up/down buttons in Streamlit with session state logging |
| HMW make the app trustworthy? | Show Wikipedia source snippet alongside each starter |

**Total ideas generated across all three sessions: 22**

---

## 3. Idea Categories & Grouping

After all sessions, ideas were grouped into five categories:

### Category A: Core Conversation Generator
Ideas 1, 8, 13, 14 — AI pipeline to generate conversation starters from user input

### Category B: Context & Intelligence Layer
Ideas 3, 4, 12, 15 — BART classification + Wikipedia enrichment

### Category C: Personalization & Tone
Ideas 6, 7, HMW-5 — Tone selection, profile-based personalization

### Category D: User Experience & Interface
Ideas 2, 13, HMW-3, HMW-7 — Streamlit, template library, source transparency

### Category E: Feedback, Learning & Gamification
Ideas 9, 11, HMW-6 — Rating loops, XP system, thumbs up/down

---

## 4. Idea Prioritization

### 4.1 Prioritization Matrix

Each idea was scored across three dimensions on a scale of 1–5:
- **Feasibility** (1 = hard/expensive, 5 = easy/cheap)
- **Impact** (1 = low user value, 5 = high user value)
- **Effort** (1 = very high effort, 5 = very low effort — inverted scale)

| Idea | Description | Feasibility | Impact | Effort | **Total** |
|---|---|---|---|---|---|
| A1 | Core GPT-2 + BART pipeline | 4 | 5 | 3 | **12** |
| A2 | Wikipedia fact-check enrichment | 5 | 4 | 4 | **13** |
| A3 | Streamlit UI | 5 | 4 | 5 | **14** |
| A4 | FastAPI backend | 5 | 4 | 4 | **13** |
| A5 | Tone selector (formal/casual) | 5 | 4 | 5 | **14** |
| A6 | Zero-shot BART event classification | 4 | 5 | 3 | **12** |
| B1 | Mobile app | 2 | 3 | 1 | **6** |
| B2 | LinkedIn browser extension | 2 | 4 | 1 | **7** |
| B3 | Real-time Eventbrite/Meetup API | 2 | 3 | 2 | **7** |
| B4 | Gamified XP system | 3 | 2 | 2 | **7** |
| B5 | Full feedback learning loop | 2 | 3 | 1 | **6** |

> **Top-scoring ideas selected for implementation:** A1, A2, A3, A4, A5, A6

---

### 4.2 MoSCoW Prioritization Method

#### ✅ MUST HAVE (Non-negotiable for v1.0)
| # | Feature | Rationale |
|---|---|---|
| M1 | User input: event name, event type, target person description | Core data needed for personalization |
| M2 | BART zero-shot classification of event theme/type | Enables intelligent, context-aware generation |
| M3 | GPT-2 text generation of 3–5 conversation starters | Primary product output |
| M4 | Wikipedia API lookup for event topic enrichment | Adds accuracy and credibility to outputs |
| M5 | FastAPI backend to serve inference pipeline | Required for clean frontend–backend separation |
| M6 | Streamlit frontend with input form and output display | Required for user interaction |

#### 🔵 SHOULD HAVE (Important but not launch-blocking)
| # | Feature | Rationale |
|---|---|---|
| S1 | Tone selector (Professional / Friendly / Curious / Humorous) | Significantly improves personalization |
| S2 | Display Wikipedia snippet as a source reference | Builds trust and transparency |
| S3 | Regenerate button to produce alternate starters | Improves usability and reduces friction |
| S4 | Copy-to-clipboard for each starter | Practical QoL feature |

#### 🟡 COULD HAVE (Nice-to-have for future versions)
| # | Feature | Rationale |
|---|---|---|
| C1 | Thumbs up/down feedback per starter | Foundation for future fine-tuning |
| C2 | Session history (last 5 generations) | Convenience for repeat users |
| C3 | Export to PDF / Share via link | Useful for event prep |
| C4 | Dark mode in Streamlit | Aesthetic improvement |

#### ❌ WON'T HAVE (Out of scope for this project)
| # | Feature | Rationale |
|---|---|---|
| W1 | Mobile app | Out of scope; web-first approach chosen |
| W2 | LinkedIn profile scraping | Privacy concerns and API restrictions |
| W3 | Real-time attendee feed integration | Too complex for current timeline |
| W4 | Reinforcement learning feedback loop | Requires production-scale data; future work |
| W5 | GPT-4 / paid LLM API | Budget constraint; open-source models preferred |

---

## 5. Final Selected Approach

### 5.1 Ranked Final Ideas by Feasibility × Impact

| Rank | Idea | Why Selected |
|---|---|---|
| 🥇 1 | **BART Zero-Shot Classification** (`facebook/bart-large-mnli`) | No fine-tuning needed; handles any event type out of the box |
| 🥈 2 | **GPT-2 Text Generation** | Open-source, local inference, no API costs, customizable prompts |
| 🥉 3 | **Wikipedia API Fact-Checking** | Free, reliable, adds factual grounding to generated starters |
| 4 | **FastAPI Backend** | Industry-standard, async-capable, easy to expose ML models as REST endpoints |
| 5 | **Streamlit Frontend** | Fastest path to a production-quality UI for ML apps |
| 6 | **Tone Selector** | High usability impact, near-zero implementation cost |

### 5.2 Final Architecture Decision

```
User Input (Streamlit UI)
        │
        ▼
FastAPI Endpoint: POST /generate
        │
        ├──► BART Zero-Shot Classifier
        │         └─ Classifies event type → label + confidence
        │
        ├──► Wikipedia API
        │         └─ Fetches fact snippet for event topic
        │
        └──► GPT-2 Text Generator
                  └─ Prompt: [event type] + [person desc] + [Wikipedia fact] + [tone]
                           → Generates 3–5 conversation starters
        │
        ▼
Streamlit: Displays starters + Wikipedia source + Tone label
```

### 5.3 Justification of Final Selection
- **No API costs:** Both BART and GPT-2 run locally via HuggingFace `transformers` library
- **No fine-tuning required:** BART's zero-shot capability works for any event theme without labeled training data
- **Factual grounding:** Wikipedia API reduces hallucination risk inherent to GPT-2
- **Fast iteration:** Streamlit enables UI changes in minutes; FastAPI supports hot-reloading
- **Scalable architecture:** FastAPI backend can be upgraded to serve a React frontend or mobile app later

---

## 6. Team Contributions to Brainstorming

## 6. Team Contributions to Brainstorming

| Team Member | Role in Brainstorming | Key Contributions |
|---|---|---|
| **Mehboob Ali Mohammed** (Team Lead) | Facilitated Session 1, structured prioritization | Problem framing, MoSCoW analysis, architecture decision |
| **Mehboob Ali Mohammed** | Facilitated Session 2, SCAMPER analysis | BART idea, SCAMPER lens, FastAPI suggestion |
| **Mehboob Ali Mohammed** | Facilitated Session 3, HMW method | Wikipedia integration idea, GPT-2 suggestion, user empathy framing |
| **Mehboob Ali Mohammed** | Active participant all sessions | Template library idea, mobile app concept, feedback loop idea |
---

## 7. Rejected Ideas & Reasoning

| Idea | Reason for Rejection |
|---|---|
| Mobile App | Outside current tech stack and timeline |
| LinkedIn Scraper | Privacy/legal issues; Terms of Service violations |
| GPT-4 API | Cost-prohibitive for a student/research project |
| Gamification (XP System) | Adds complexity without direct value to core use case |
| Eventbrite/Meetup API | Real-time data integration adds scope risk |
| Full Reinforcement Learning Loop | Requires production data volume and infrastructure |

---

## 8. Summary & Next Steps

### Summary
Through three structured brainstorming sessions using Classic Brainstorm, SCAMPER, and HMW methodologies, the team generated **22 distinct ideas**. These were systematically evaluated using a **Prioritization Matrix** and narrowed down using the **MoSCoW method**. The final selected stack — **BART + GPT-2 + Wikipedia API + FastAPI + Streamlit** — offers the highest combined feasibility and impact within the project's constraints.

### Next Steps
- [ ] Finalize problem statement and empathy map (Week 2)
- [ ] Set up project repository structure (Week 2)
- [ ] Implement FastAPI skeleton with BART endpoint (Week 3)
- [ ] Integrate Wikipedia API call into pipeline (Week 3)
- [ ] Build Streamlit UI and connect to FastAPI (Week 4)
- [ ] End-to-end integration testing (Week 5)
- [ ] Presentation and demo preparation (Week 6)

---

*Document prepared by the Personalized Networking Assistant Team — July 2026*
