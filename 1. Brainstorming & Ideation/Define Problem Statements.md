# Define Problem Statements
### Personalized Networking Assistant — AI-Powered Conversation Starter Web App

---

## 📋 Document Information

| Field | Details |
|---|---|
| **Project Name** | Personalized Networking Assistant |
| **Document Type** | Problem Statement Definition |
| **Developer** | mehboob ali mohammed |
| **Team Members** | mehboob ali mohammed |
| **Date** | July 2026 |
| **Version** | 1.0 |

---

## 1. Current Situation (As-Is State)

### 1.1 Background
Every year, millions of professionals — from freshly enrolled graduate students to seasoned mid-career engineers — attend networking events: university career fairs, technology conferences, alumni meets, hackathons, and industry summits. The universal expectation at these events is that attendees will meet strangers, exchange ideas, form relationships, and create opportunities.

Yet a fundamental and well-documented barrier exists at the very beginning of these interactions: **the opening line**. Decades of social psychology research confirm that first impressions are formed in the first 7 seconds of an interaction, and that the quality of an opening conversation determines whether a relationship will progress or stall.

### 1.2 Observed User Behavior
Through team observation and secondary research, we identified the following typical user behavior at networking events:

1. **Pre-event anxiety:** Most attendees report nervousness or stress before approaching strangers
2. **Generic openers:** The vast majority default to predictable questions — *"What do you do?"*, *"How do you know the host?"*, *"Have you been to this conference before?"*
3. **Conversation stalling:** Without relevant, specific content to discuss, conversations plateau within 90 seconds
4. **Opportunity loss:** Attendees leave events without making meaningful connections despite genuine intent
5. **Post-event regret:** A common sentiment is "I wish I had said something more interesting"

### 1.3 Current Tools & Their Limitations
| Existing Tool | What It Does | What It Misses |
|---|---|---|
| LinkedIn | Profile viewing, connection requests | Does not generate conversation content |
| Eventbrite / Meetup | Event discovery, RSVP | No contextual conversation prep |
| Google / Wikipedia | Manual research | Requires user to synthesize information manually |
| Generic networking advice blogs | Static tips | Not personalized to the specific event or person |
| ChatGPT (generic) | Can generate openers if prompted | Requires user to know how to prompt well; no structured workflow |

> **Key Insight:** No existing tool combines **event context understanding**, **AI-powered language generation**, and **real-time fact enrichment** into a single, user-friendly interface designed specifically for networking conversation preparation.

---

## 2. Formal Problem Statement

### 2.1 Primary Problem Statement (HMW Format)
> **"How might we help professionals generate personalized, AI-powered conversation starters for any networking event, so that they can make meaningful connections more confidently?"**

### 2.2 Supporting Problem Statements

| # | Sub-Problem Statement |
|---|---|
| P1 | How might we understand the theme and context of any networking event without requiring manual categorization by the user? |
| P2 | How might we ensure that AI-generated conversation starters are factually accurate and not based on model hallucinations? |
| P3 | How might we allow users to personalize starters based on the specific person they want to approach? |
| P4 | How might we deliver an end-to-end solution that requires zero technical knowledge to use? |
| P5 | How might we make the system work for any professional domain — tech, healthcare, finance, arts — without domain-specific training? |

### 2.3 Problem Statement Validation
The primary problem statement was validated against the following criteria:

| Criterion | Status | Notes |
|---|---|---|
| **Human-centered** | ✅ Validated | Centers on the user's emotional and practical struggle |
| **Broad enough to allow creative solutions** | ✅ Validated | Doesn't prescribe a specific technology solution |
| **Narrow enough to be actionable** | ✅ Validated | Scoped to networking events specifically |
| **Avoids assumed solutions** | ✅ Validated | Asks "how might we" rather than "build X" |
| **Backed by observed pain** | ✅ Validated | Consistent with team research and user observations |

---

## 3. Target Users

### 3.1 Primary User Personas

#### Persona 1: The Graduate Student
| Attribute | Detail |
|---|---|
| **Name** | Sumiya (representative) |
| **Age** | 22–27 |
| **Context** | Attending university career fairs, research conferences, hackathons |
| **Goal** | Secure internships, research collaborations, mentorships |
| **Tech Comfort** | High — comfortable with web apps |
| **Networking Experience** | Low to medium — limited professional history |

#### Persona 2: The Early-Career Professional
| Attribute | Detail |
|---|---|
| **Name** | Jagan (representative) |
| **Age** | 25–32 |
| **Context** | Industry conferences, meetups, company onboarding events |
| **Goal** | Build a professional network, find mentors, explore job opportunities |
| **Tech Comfort** | High |
| **Networking Experience** | Medium — has attended events but still finds initiation difficult |

#### Persona 3: The Introverted Specialist
| Attribute | Detail |
|---|---|
| **Name** | mehboob ali mohammed (representative) |
| **Age** | 28–40 |
| **Context** | Domain-specific conferences (AI, biotech, legal, finance) |
| **Goal** | Meet peers, find collaborators, present work |
| **Tech Comfort** | Variable |
| **Networking Experience** | Low — prefers deep work to social interaction |

### 3.2 Secondary User Personas
- **Career counselors and coaches** who prepare students for events
- **Event organizers** who want to add a conversation-starter feature to their platform
- **HR professionals** preparing employees for industry conferences

---

## 4. Pain Points

### 4.1 Detailed Pain Points with Evidence

#### Pain Point 1: Cognitive Overload at the Moment of Introduction
**Description:** When a professional stands in front of a new contact, they must simultaneously process social cues, recall relevant facts, maintain body language, and formulate a conversational opener — all in real time.
**Evidence:** Research in social cognition shows that spontaneous social interaction is one of the highest cognitive-load activities humans perform (Lieberman, 2013).
**Impact:** Leads to generic, forgettable openers that fail to establish rapport.

---

#### Pain Point 2: Lack of Event-Specific Knowledge
**Description:** Attendees often do not have enough background on the event's topic or theme to ask informed, impressive questions.
**Example:** A computer science student attending a "Women in Fintech" conference may not know enough about current fintech trends to open a conversation beyond small talk.
**Impact:** Conversations remain surface-level; attendees lose credibility as informed professionals.

---

#### Pain Point 3: Fear of Saying Something Inaccurate or Embarrassing
**Description:** The risk of confidently stating a fact that turns out to be wrong is a strong inhibitor, particularly for students and early-career professionals.
**Evidence:** Survey data from LinkedIn's "State of Professional Networking" reports that 50% of professionals avoid initiating conversations at events due to fear of embarrassment.
**Impact:** Prevents conversation initiation entirely; lost networking opportunities.

---

#### Pain Point 4: One-Size-Fits-All Generic Advice
**Description:** Existing resources (books, blog posts, YouTube videos) provide generic networking tips that are not tailored to the event type, industry, or the specific person being approached.
**Example:** The advice "ask open-ended questions" is theoretically correct but practically unhelpful without specific content.
**Impact:** Users feel unprepared even after consuming large amounts of networking advice.

---

#### Pain Point 5: No Real-Time Assistance Tool for Professionals
**Description:** While AI tools like ChatGPT exist, they require the user to construct a detailed prompt from scratch, understand prompt engineering, and evaluate the outputs critically — skills most users do not have in a stressful pre-event environment.
**Impact:** The cognitive barrier to using general-purpose AI tools is too high for networking event preparation.

---

## 5. Gap Analysis

### 5.1 What Existing Tools Are Missing

| Gap | Description | Our Solution |
|---|---|---|
| **Structured Input Pipeline** | No tool accepts event + person description together as input | Our Streamlit form captures event name, event type, person description, and desired tone |
| **Event Theme Intelligence** | No tool automatically understands the type/theme of an event | BART zero-shot classification categorizes events without pre-training |
| **AI-Generated, Event-Specific Content** | No tool generates novel conversation starters grounded in the event context | GPT-2 pipeline generates 3–5 unique starters per query |
| **Real-Time Fact Enrichment** | No tool verifies or enriches AI outputs with factual data | Wikipedia API fetches and injects a verified fact snippet into every generation |
| **Integrated, No-Code Interface** | No tool combines all of the above in a zero-friction UI | Streamlit frontend: fill form → click button → see results |
| **Tone/Style Customization** | No tool lets users select formal vs. casual communication style | Tone selector (Professional, Friendly, Curious, Humorous) embedded in UI |

---

## 6. Project Scope

### 6.1 In-Scope Items
| # | Feature | Description |
|---|---|---|
| 1 | Web-based input form | Accept event name, event description, target person description, tone preference |
| 2 | BART event classification | Zero-shot classification of event theme using `facebook/bart-large-mnli` |
| 3 | Wikipedia enrichment | Fetch top-paragraph Wikipedia summary for the detected event topic |
| 4 | GPT-2 generation | Generate 3–5 conversation starters using a structured prompt |
| 5 | FastAPI backend | RESTful API exposing the full inference pipeline |
| 6 | Streamlit frontend | Interactive, styled web interface consuming the FastAPI endpoint |
| 7 | Tone modulation | Prompt engineering to adjust formality and style of outputs |
| 8 | Source transparency | Display Wikipedia snippet alongside generated starters |

### 6.2 Out-of-Scope Items
| # | Feature | Reason |
|---|---|---|
| 1 | Mobile application | Outside current tech stack and timeline |
| 2 | User authentication and profile management | Not required for MVP; adds complexity |
| 3 | LinkedIn or social media integration | Privacy concerns and API ToS limitations |
| 4 | Real-time event feed data (Eventbrite, Meetup) | Integration complexity exceeds current scope |
| 5 | Model fine-tuning or training from scratch | Requires compute resources beyond project budget |
| 6 | Persistent database for conversation history | Out of scope for v1.0; considered for v2.0 |
| 7 | Multi-language support | English-only for v1.0 |
| 8 | Paid LLM APIs (GPT-4, Claude) | Budget constraint; open-source models preferred |

---

## 7. Success Metrics

### 7.1 Functional Success Metrics (Technical)
| Metric | Target | Measurement Method |
|---|---|---|
| BART classification accuracy | ≥ 85% on 20 test event descriptions | Manual evaluation against ground truth labels |
| Wikipedia API success rate | ≥ 90% successful topic fetch | Logged API calls during testing |
| GPT-2 generation time | ≤ 10 seconds per request | Timed via FastAPI response time logging |
| API uptime during demo | 100% | Manual monitoring during presentation |
| End-to-end pipeline success rate | ≥ 95% (no crashes on valid input) | Integration test results |

### 7.2 Quality Success Metrics (Output Quality)
| Metric | Target | Measurement Method |
|---|---|---|
| Starter relevance to event | ≥ 4/5 rating by 3 evaluators | Team evaluation rubric |
| Starter naturalness / fluency | ≥ 3.5/5 rating by evaluators | Team evaluation rubric |
| Factual accuracy of Wikipedia enrichment | 100% (directly from Wikipedia) | Source verification |
| Tone appropriateness | ≥ 80% tone-label match in human eval | Manual annotation |

### 7.3 User Experience Success Metrics
| Metric | Target | Measurement Method |
|---|---|---|
| Task completion rate (input → output) | 100% by first-time users in demo | Observation |
| Time to first generated output | ≤ 15 seconds from form submission | Stopwatch timing |
| User satisfaction rating | ≥ 4/5 from 5 demo users | Post-demo survey |
| UI clarity / no confusion | 0 UX blockers during demo | Observation log |

---

## 8. Problem–Solution Fit Summary

| Problem | Our Solution |
|---|---|
| Professionals don't know how to start conversations | GPT-2 generates 3–5 ready-to-use starters |
| Generic openers fail to impress | BART event classification enables context-aware generation |
| Fear of factual inaccuracy | Wikipedia API grounds outputs in verified facts |
| General AI tools are hard to prompt correctly | Streamlit form provides a structured, no-code workflow |
| No tool works across all event types and industries | Zero-shot BART needs no domain-specific training data |

---

*Document prepared by the Personalized Networking Assistant Team — July 2026*
