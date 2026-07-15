# Demonstration of Proposed Features: Personalized Networking Assistant

## 1. Introduction & Overview
This document serves as the official, step-by-step feature demonstration script and verification guide for the **Personalized Networking Assistant**. The application integrates a responsive **Streamlit** multi-tab user interface with a high-performance **FastAPI** backend, utilizing deep learning models (**BART** for theme extraction and **GPT-2** for conversation starter generation) alongside real-time **Wikipedia API** integration for fact-checking.

During the project demonstration, this script will be executed live to showcase all 12 proposed features, verifying end-to-end functionality, real-time AI inference, persistence, and interactive analytics.

---

## 2. Feature Demonstration Checklist (All 12 Features)

Before and during the demonstration, the presenter will verify and check off each of the 12 core features across our technical stack:

| # | Feature Name | Core Module / Technology | Verified Status |
| :---: | :--- | :--- | :---: |
| **1** | **Event Input Analysis** | Streamlit UI (`streamlit_app.py`) + Pydantic Validation | [x] Operational |
| **2** | **Theme Extraction via BART** | NLP Service (`nlp_service.py`) / Hugging Face BART | [x] Operational |
| **3** | **Conversation Starter Generation** | Generation Service (`generation_service.py`) / GPT-2 | [x] Operational |
| **4** | **Custom Tone/Style Selection** | Streamlit Sidebar + Prompt Engineering Engine | [x] Operational |
| **5** | **Interactive Feedback & Rating** | Streamlit Widgets + `/feedback` API Endpoint | [x] Operational |
| **6** | **Real-Time Wikipedia Fact-Checking** | Fact Checker (`fact_checker.py`) + Wikipedia REST API | [x] Operational |
| **7** | **Confidence Scoring & Verification** | TF-IDF / Lexical Similarity Engine | [x] Operational |
| **8** | **Session History & Persistence** | History Module (`history.py`) + Local JSON Storage | [x] Operational |
| **9** | **Interactive Analytics Dashboard** | Dashboard Module (`dashboard.py`) + Plotly/Altair Charts | [x] Operational |
| **10** | **Multi-Tab Streamlit Interface** | Streamlit Tabs (`st.tabs`) Component Architecture | [x] Operational |
| **11** | **Real-Time Validation & Error Handling** | FastAPI Exception Handlers + Streamlit Alerts | [x] Operational |
| **12** | **RESTful API & Swagger UI Docs** | FastAPI (`app.py`) + OpenAPI / Swagger UI | [x] Operational |

---

## 3. Step-by-Step Walkthrough & Demo Script

The presenter (**mehboob ali mohammed** driving complete demonstration) will follow this exact 10-step sequence during the live presentation.

### Step 1: Open Streamlit Application at `http://localhost:8501`
* **Action:** Open Google Chrome or Mozilla Firefox and navigate to `http://localhost:8501`.
* **Presenter Script:** *"Welcome to the Personalized Networking Assistant. As you can see on our home screen, we have a clean, intuitive multi-tab interface built with Streamlit. On the left sidebar, we have configuration controls for tailoring AI outputs, while our main panel is divided into four distinct operational tabs: Generator, Fact-Check, History, and Dashboard."*
* **Screenshot Description:** Displays the full application layout. The top banner reads **"🤝 Personalized Networking Assistant"** with a subtitle explaining its AI-driven capabilities. The sidebar on the left displays model settings, tone selectors (Professional, Casual, Enthusiastic, Technical), and an API connectivity indicator showing a green dot labeled **"FastAPI Backend: Online (Port 8000)"**. The main area shows four prominent tabs: `⚡ Generator`, `🔍 Fact-Check`, `📜 History`, and `📊 Dashboard`.

---

### Step 2: Enter Event Description & User Interests
* **Action:** In the `⚡ Generator` tab, enter the following details into the input form:
  * **Event Description:** `AI for Sustainable Cities: Annual Summit gathering urban planners, data scientists, and IoT engineers to discuss smart grids, automated traffic management, and green infrastructure.`
  * **User Interests / Background:** `Machine Learning, Renewable Energy Systems, Edge Computing.`
  * **Tone Selection (Sidebar):** Select `Professional & Technical`.
* **Presenter Script:** *"Let's imagine we are attending an annual technology summit focused on 'AI for Sustainable Cities.' We input our event description and specify our personal background in Machine Learning and Renewable Energy. We also select a 'Professional & Technical' tone from our sidebar to ensure the generated starters match the caliber of the event."*
* **Screenshot Description:** Shows the filled-out text area for "Event Description" and text input for "Your Interests". The sidebar radio button for Tone is highlighted on "Professional & Technical". A prominent primary button labeled **"🚀 Analyze & Generate Starters"** is visible at the bottom of the form.

---

### Step 3: Click Generate — Show BART Extracting Themes
* **Action:** Click the **"🚀 Analyze & Generate Starters"** button. Observe the loading spinner and subsequent theme extraction display.
* **Presenter Script:** *"Upon clicking Generate, our Streamlit frontend sends an asynchronous REST request to our FastAPI backend. Instantly, our fine-tuned BART model analyzes the text using zero-shot classification and sequence summarization to extract the core thematic pillars of this summit."*
* **Screenshot Description:** A Streamlit status container shows a spinner with the text *"🧠 Analyzing event themes with BART and generating starters with GPT-2..."*. Below it, a newly rendered section titled **"🎯 Extracted Event Themes (BART NLP Engine)"** displays three colored tags/badges: `[Smart Grids & Energy]`, `[Automated Traffic Management]`, and `[Green IoT Infrastructure]`, alongside a confidence score metric showing `Theme Extraction Confidence: 94.2%`.

---

### Step 4: Show GPT-2 Generated Conversation Starters
* **Action:** Scroll down to view the generated conversation starters rendered in expandable cards.
* **Presenter Script:** *"Guided by the extracted themes and our selected technical tone, our GPT-2 generation service constructs tailored, high-impact conversation starters. Notice how each starter bridges the user's background in ML and Renewable Energy directly with the summit's topics."*
* **Screenshot Description:** Displays a section titled **"💬 Personalized Conversation Starters (GPT-2 Engine)"**. Four styled cards are presented:
  * **Starter 1 (Icebreaker):** *"I noticed this session focuses heavily on smart grids. How is your team integrating machine learning to optimize real-time load balancing in renewable energy systems?"*
  * **Starter 2 (Technical Deep-Dive):** *"With automated traffic management scaling up, what edge computing architectures are you finding most effective for processing live IoT sensor feeds without high cloud latency?"*
  * **Starter 3 (Collaborative/Exploratory):** *"As someone working at the intersection of ML and green infrastructure, I’m curious—what do you see as the biggest bottleneck in deploying AI for municipal water management?"*
  * **Starter 4 (Casual Professional):** *"Fascinating keynote on sustainable urban planning today! Are you currently leveraging any predictive AI models for city emissions tracking in your projects?"*

---

### Step 5: Click Thumbs Up & Rate 4 Stars on a Starter
* **Action:** On **Starter 1**, click the **Thumbs Up (👍)** button and select **4 Stars (⭐⭐⭐⭐)** on the interactive rating widget. Click **"Submit Feedback"**.
* **Presenter Script:** *"To continuously improve our system and track user preferences, every starter features an interactive feedback mechanism. Let's give this first smart grid starter a thumbs up and rate it 4 stars. When we submit, this feedback is sent to our backend `/feedback` endpoint and logged for real-time analytics."*
* **Screenshot Description:** Shows the interactive feedback row beneath Starter 1. The thumbs-up button is highlighted in green (`👍 Helpful`). A star rating component shows 4 out of 5 stars illuminated in gold. A green toast notification in the bottom right corner displays: *"✅ Feedback recorded successfully! Thank you for helping us improve."*

---

### Step 6: Switch to Fact-Check Tab — Search 'Blockchain in Healthcare'
* **Action:** Click on the `🔍 Fact-Check` tab at the top of the main panel. In the search input box, type `blockchain in healthcare` and click **"Verify Claim via Wikipedia"**.
* **Presenter Script:** *"In networking environments, discussing emerging tech often involves complex factual claims. To prevent spreading misinformation or getting caught unprepared, we built a real-time Fact-Checking engine. Let's switch to the Fact-Check tab and verify a discussion topic: 'blockchain in healthcare'."*
* **Screenshot Description:** The UI displays the `🔍 Fact-Check` tab active. A search input box contains the string `"blockchain in healthcare"`. A blue action button labeled **"🔍 Verify Claim via Wikipedia"** is being clicked. A brief progress bar indicates API querying.

---

### Step 7: Show Wikipedia Result with Confidence Rating
* **Action:** View the populated fact-check summary, Wikipedia citation link, and lexical confidence score.
* **Presenter Script:** *"Our backend queries the Wikipedia REST API, retrieves the authoritative summary, and calculates a lexical confidence rating comparing our query against verified encyclopedic entities. Here we see a detailed summary of blockchain applications in medical record management, accompanied by an 88.5% High Confidence rating and a direct citation link."*
* **Screenshot Description:** A structured result box displays:
  * **Status Badge:** `🟢 High Confidence Match (Score: 0.885)`
  * **Article Title:** **Blockchain in healthcare**
  * **Summary Text:** *"Blockchain technology in healthcare is utilized to manage electronic health records (EHRs), ensure drug supply chain integrity, and maintain data security in biomedical research. Decentralized ledgers allow secure, interoperable data exchange among hospitals, insurers, and patients while maintaining strict HIPAA compliance..."*
  * **Source Citation:** A hyperlink reading `📖 Read full verified article on Wikipedia ↗` pointing to the official Wikipedia URL.

---

### Step 8: Go to History Tab — Show Saved Sessions
* **Action:** Click on the `📜 History` tab. Show the chronological list of previously generated networking sessions.
* **Presenter Script:** *"All our networking preparations are automatically saved to persistent local storage. In the History tab, users can review past event analyses, recall generated conversation starters from previous conferences, and check which topics they previously favored."*
* **Screenshot Description:** Displays a table and accordion list of historical sessions. Each entry shows a timestamp (e.g., `2026-07-07 14:30 IST`), Event Title (`AI for Sustainable Cities`), Extracted Themes badges, and an expander arrow reading `View 4 Starters & Feedback`. An action button at the top right reads **"🗑️ Export History (JSON/CSV)"**.

---

### Step 9: Visit Dashboard — Show Analytics
* **Action:** Click on the `📊 Dashboard` tab. Walk through the interactive visual charts depicting user engagement and topic distributions.
* **Presenter Script:** *"Now let's explore the Analytics Dashboard, presented by Satvika. This tab aggregates all historical sessions and feedback into interactive Plotly charts. We can see our most frequently explored networking themes, average feedback ratings across different tones, and total sessions generated over time."*
* **Screenshot Description:** Displays three interactive charts:
  1. **Top Networking Themes (Bar Chart):** Shows `Machine Learning` (24 sessions), `Renewable Energy` (18 sessions), `IoT & Edge` (15 sessions), and `FinTech` (12 sessions).
  2. **Tone Preference vs. Average Rating (Radar/Line Chart):** Indicates that `Professional & Technical` tone receives the highest average user rating (`4.6 / 5.0`).
  3. **Activity Timeline (Area Chart):** A smooth curve showing session generation activity over the last 14 days.

---

### Step 10: Show Swagger UI at `http://localhost:8000/docs`
* **Action:** Open a new browser tab and navigate to `http://localhost:8000/docs`. Expand the `/generate` and `/fact-check` endpoints and execute a live test request.
* **Presenter Script:** *"Finally, to demonstrate the enterprise-grade backend architecture built by Sumiya Zainab, we visit our interactive OpenAPI Swagger documentation at port 8000. Our FastAPI backend exposes clean, fully documented RESTful endpoints with strict Pydantic validation. Let's execute a live test directly through Swagger UI."*
* **Screenshot Description:** Displays the standard Swagger UI dark/light interface titled **"Personalized Networking Assistant API v1.0.0"**. Five endpoints are listed:
  * `POST /api/v1/analyze` (Analyze event text)
  * `POST /api/v1/generate` (Generate conversation starters)
  * `POST /api/v1/fact-check` (Verify claims via Wikipedia)
  * `GET /api/v1/history` (Retrieve session history)
  * `POST /api/v1/feedback` (Submit user rating)
  * The `/api/v1/generate` endpoint is expanded, showing the Pydantic request body schema and a successful `200 OK` response payload containing generated JSON starters.

---

## 4. Live API Demonstration Using Swagger UI

During Step 10, the presenter will execute the following live test in Swagger UI to prove backend decoupling and robustness:

### 4.1 Endpoint: `POST /api/v1/generate`
* **Request Payload (JSON):**
```json
{
  "event_description": "Global FinTech & Blockchain Symposium focusing on decentralized finance and AI-driven fraud detection.",
  "user_interests": "Quantitative Trading, Smart Contracts, Deep Learning",
  "tone": "Technical",
  "num_starters": 3
}
```
* **Expected Response (200 OK):**
```json
{
  "status": "success",
  "timestamp": "2026-07-07T22:15:00Z",
  "extracted_themes": [
    "Decentralized Finance (DeFi)",
    "AI Fraud Detection",
    "Smart Contracts"
  ],
  "starters": [
    {
      "id": "gen_001",
      "text": "How is your organization adapting quantitative trading algorithms to handle the liquidity shifts in decentralized finance protocols?",
      "theme": "Decentralized Finance (DeFi)",
      "confidence": 0.91
    },
    {
      "id": "gen_002",
      "text": "With AI-driven fraud detection becoming vital, what deep learning architectures are you implementing to detect anomaly patterns in real-time smart contract executions?",
      "theme": "AI Fraud Detection",
      "confidence": 0.94
    },
    {
      "id": "gen_003",
      "text": "I’m fascinating by the intersection of deep learning and smart contracts. Are you seeing significant progress in automated code auditing using LLMs?",
      "theme": "Smart Contracts",
      "confidence": 0.89
    }
  ]
}
```

---

## 5. Demo Backup Plan (If Internet is Unavailable)

To mitigate network risks or Wi-Fi failures during the live presentation, the team has implemented a robust **Three-Tier Backup Strategy**:

```
+-----------------------------------------------------------------------------------+
|                            THREE-TIER BACKUP STRATEGY                             |
+-----------------------------------------------------------------------------------+
| TIER 1: Offline Local AI Cache (Default Failover)                                 |
| • Hugging Face BART and GPT-2 weights pre-downloaded to ~/.cache/huggingface      |
| • Models execute 100% locally from RAM/CPU without internet connectivity          |
+-----------------------------------------------------------------------------------+
| TIER 2: Mocked Wikipedia Fallback & Local Knowledge Dump                          |
| • Fact-Checker automatically catches requests.exceptions.ConnectionError          |
| • Switches to local JSON knowledge dictionary (1,000 top tech/science terms)      |
+-----------------------------------------------------------------------------------+
| TIER 3: Pre-Recorded High-Definition Video Walkthrough                            |
| • 1080p 60fps video walkthrough of all 10 demo steps saved on Desktop & USB drive |
| • Presenter narrates over video if local host hardware encounters critical crash  |
+-----------------------------------------------------------------------------------+
```

### 5.1 Step-by-Step Execution of Backup Plan
1. **Model Cache Verification:** 24 hours prior to the demo, the team runs `python -c "from transformers import AutoModel; AutoModel.from_pretrained('facebook/bart-large-mnli'); AutoModel.from_pretrained('gpt2')"` to guarantee weights are cached locally.
2. **Offline Fact-Check Handling:** In `fact_checker.py`, an exception block intercepts connection failures:
   ```python
   try:
       result = wikipedia.summary(query, sentences=3)
   except (requests.exceptions.ConnectionError, wikipedia.exceptions.WikipediaException):
       # Fallback to local offline dictionary
       result = LOCAL_OFFLINE_DICTIONARY.get(query.lower(), "Offline Mode: Live Wikipedia access unavailable. Displaying cached domain knowledge.")
   ```
3. **Emergency Video Walkthrough:** If local server ports fail to bind due to host OS restrictions, **Tejesh Velivela** will instantly open `C:\Users\ADMIN\Desktop\NetAssist_Demo_2026.mp4` using VLC Media Player, allowing the team to deliver their scripted presentation seamlessly without loss of continuity or professionalism.
