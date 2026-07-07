# Personalized Networking Assistant: User Guide

![Screenshot Placeholder: Banner](../images/banner.png)

---

## Table of Contents
1. [Introduction & Purpose](#1-introduction--purpose)
2. [Important Note on Authentication & Privacy (MVP Architecture)](#2-important-note-on-authentication--privacy-mvp-architecture)
3. [Home Page & Navigation Overview](#3-home-page--navigation-overview)
4. [Feature Walkthroughs](#4-feature-walkthroughs)
   - [4.1 Generate Starters Tab](#41-generate-starters-tab)
   - [4.2 Fact-Checker Tab](#42-fact-checker-tab)
   - [4.3 History Browser Tab](#43-history-browser-tab)
   - [4.4 Analytics Dashboard Tab](#44-analytics-dashboard-tab)
5. [Step-by-Step Usage Guide](#5-step-by-step-usage-guide)
6. [Frequently Asked Questions (FAQs)](#6-frequently-asked-questions-faqs)
7. [User Troubleshooting Guide](#7-user-troubleshooting-guide)

---

## 1. Introduction & Purpose

Professional networking at academic conferences, industry summits, and corporate meetups can be daunting. Initiating meaningful conversations, breaking the ice with domain experts, and ensuring that technical topics are accurately discussed requires significant real-time cognitive effort. Many professionals struggle with:
- **Generic Icebreakers:** Relying on superficial conversation starters that fail to engage domain specialists.
- **Context Overload:** Navigating complex event descriptions and extracting relevant networking themes on the fly.
- **Information Verification:** Making references to recent technical developments, organizations, or concepts without an immediate way to verify their accuracy.

The **Personalized Networking Assistant** is an advanced AI/ML-powered conversational application designed to bridge this gap. Built for competition-grade performance and real-world utility, the application combines **Zero-Shot Natural Language Processing (NLP)** and **Generative Language Models** to analyze event contexts, extract core professional themes, and generate tailored, high-impact conversation starters. Furthermore, it integrates an automated **Fact-Checking Engine** powered by Wikipedia's REST API to verify technical claims and entities in real time.

Whether you are a researcher attending an IEEE conference, a software engineer at a tech summit, or a student preparing for a career fair, the Personalized Networking Assistant serves as your real-time co-pilot for meaningful professional interactions.

---

## 2. Important Note on Authentication & Privacy (MVP Architecture)

> [!IMPORTANT]
> **No Login Required for Local MVP:** As a Minimum Viable Product (MVP) designed for local execution, academic evaluation, and AI/ML competition demonstration, the Personalized Networking Assistant **does not require user login, account creation, or cloud authentication**.

### Why is there no login?
In this architectural release, our primary design mandate was to demonstrate high-performance AI/ML inference (using Hugging Face `BART` and `GPT-2`), low-latency API orchestration (via FastAPI), and seamless user interaction (via Streamlit) without requiring external third-party cloud infrastructure or database servers.

### How are sessions and data handled?
- **Local Atomic Persistence:** All user interactions, generated conversation starters, fact-checking queries, and sentiment feedback are stored locally on your machine in atomic JSON format (`data/history.json` and `data/feedback.json`).
- **Complete Data Privacy:** Because the application runs entirely on your local machine (or within your local Docker containers), your networking notes, event descriptions, and historical conversations never leave your environment.
- **Future Architectural Roadmap:** While the current architectural constraint eliminates multi-user account management and cloud synchronization, future production releases will introduce OAuth2/JWT authentication, PostgreSQL cloud database integration, and automated email/calendar sync.

---

## 3. Home Page & Navigation Overview

When you launch the application and navigate to `http://localhost:8501` in your web browser, you are greeted by the clean, responsive **Streamlit Interface**. The interface is structured around a **Multi-Page Tabbed Layout** that separates functionality into logical workflows while maintaining a cohesive user experience.

![Screenshot Placeholder: Home Page Overview](../images/banner.png)

### Layout & Elements
| UI Component | Location | Description |
| :--- | :--- | :--- |
| **Header Banner** | Top of Page | Displays the application title, version, and quick status indicators connecting to the backend API (`http://localhost:8000`). |
| **Navigation Tabs** | Top Horizontal Bar | Allows seamless switching between the four core modules: **Generate Starters**, **Fact-Check**, **History**, and **Analytics Dashboard**. |
| **Sidebar Controls** | Left Panel | Provides global application controls, API endpoint status monitoring, documentation links, and quick-reset buttons for local session state. |
| **Main Content Area** | Center | Dynamically renders interactive forms, AI-generated outputs, verification cards, and data visualizations based on the active tab. |

---

## 4. Feature Walkthroughs

### 4.1 Generate Starters Tab
The **Generate Starters** module is the core AI engine of the application. It takes raw event descriptions and transforms them into customized, engaging conversation starters.

![Screenshot Placeholder: Generate Starters Tab](../images/generate.png)

#### How It Works:
1. **Event Analysis & Theme Extraction:** When you input an event description (e.g., *"International Conference on Autonomous Systems and Robotics, focusing on reinforcement learning and ethical AI"*), the frontend sends a request to the backend `/api/v1/analyze-event` endpoint. The backend leverages Hugging Face's `facebook/bart-large-mnli` model to perform **Zero-Shot Classification**, automatically extracting key professional themes (e.g., *Artificial Intelligence, Robotics, Ethics*).
2. **Conversation Starter Generation:** Once themes are identified, the system calls `/api/v1/generate-conversation`. The local `gpt2` (124M parameter) model generates natural, context-aware icebreakers tailored to those specific themes.
3. **Interactive Feedback & Rating System:** Every generated conversation starter is accompanied by interactive feedback controls:
   - **Thumbs Up (👍) / Thumbs Down (👎):** Capture immediate qualitative sentiment regarding the relevance and tone of the starter.
   - **1-5 Star Ratings:** Allow granular scoring of the generated text. This feedback is sent directly to `/api/v1/feedback` and stored in `data/feedback.json` for model evaluation and analytics.

---

### 4.2 Fact-Checker Tab
In technical networking, accuracy is paramount. The **Fact-Checker** module acts as an automated verification assistant, ensuring that historical facts, technical terminology, and organizational details mentioned in conversations are reliable.

![Screenshot Placeholder: Fact-Checker Tab](../images/factcheck.png)

#### How It Works:
1. **Query Submission:** Users enter a statement, topic, or entity (e.g., *"Transformer architecture was introduced by Google in 2017"* or *"Quantum computing supremacy"*).
2. **Wikipedia REST API Verification:** The backend `/api/v1/fact-check` endpoint queries the Wikipedia REST API (`https://en.wikipedia.org/api/rest_v1/page/summary/{query}`) using a custom, polite User-Agent string.
3. **Confidence Scoring Engine:** The system evaluates the retrieved summary and assigns an automated confidence score:
   - **High Confidence (`high`):** Direct exact match with comprehensive encyclopedic consensus and high lexical overlap.
   - **Medium Confidence (`medium`):** Partial match or related concept identified; useful for general context but requires minor human verification.
   - **Low Confidence (`low`):** Ambiguous topic or limited verification sources found.
4. **Defensive Fallback Mechanism:** If the external Wikipedia API experiences network timeouts, rate limiting, or connectivity issues, the application gracefully activates its fallback mode. It informs the user of the network condition without crashing, preserving offline usability.

---

### 4.3 History Browser Tab
The **History Browser** provides a comprehensive audit trail of all your networking preparation sessions. Never lose a great conversation starter or fact-check result.

![Screenshot Placeholder: History Browser Tab](../images/history.png)

#### Features:
- **Chronological Review:** View past event analyses and generated starters ordered by ISO-8601 UTC timestamps.
- **Search & Filter:** Use the real-time search bar to filter historical records by keyword, theme, or event name.
- **Pagination Controls:** Navigate smoothly through large interaction histories without UI lag or memory clutter.
- **Atomic Persistence Guarantee:** Powered by the backend `/api/v1/history` endpoint reading from `data/history.json`, ensuring your data remains uncorrupted even during unexpected shutdowns.

---

### 4.4 Analytics Dashboard Tab
The **Analytics Dashboard** offers visual insights into your networking preparation habits and AI model performance.

![Screenshot Placeholder: Analytics Dashboard Tab](../images/dashboard.png)

#### Visual Metrics Included:
- **Interaction Volume Chart:** A bar chart illustrating the number of conversation starters generated over time.
- **Feedback Distribution:** Visual breakdown of positive (👍) versus negative (👎) feedback received across different networking themes.
- **Average Star Rating:** Real-time calculation of overall user satisfaction with the `GPT-2` generated icebreakers.
- **Theme Frequency Analysis:** Identifies the most common professional topics and domains you have explored.

---

## 5. Step-by-Step Usage Guide

Follow these numbered instructions to prepare for your next professional event using the Personalized Networking Assistant:

### Step 1: Launch the Application
1. Ensure your Docker containers or local virtual environment is running (see [Developer Guide](Developer_Guide.md) for setup).
2. Open your web browser and navigate to `http://localhost:8501`.
3. Verify that the top status bar indicates **"Backend API: Connected (Port 8000)"**.

### Step 2: Extract Themes & Generate Icebreakers
1. Click on the **"Generate Starters"** tab in the top navigation bar.
2. In the **Event Description** text area, paste or type the details of the event you are attending.
   *Example input: "Annual IEEE Symposium on Deep Learning and Computer Vision. Key topics include generative adversarial networks, edge AI computing, and autonomous driving safety standards."*
3. Click the primary **"Analyze Event & Generate Starters"** button.
4. Wait a few seconds while the backend executes `BART` zero-shot classification and `GPT-2` text generation.
5. Review the extracted themes displayed as colored badges (e.g., `Deep Learning`, `Computer Vision`, `Edge AI`).
6. Read through the generated conversation starters displayed in the results cards below.

### Step 3: Rate the Generated Starters
1. For each generated conversation starter card, evaluate its relevance and tone.
2. Click the **👍 (Thumbs Up)** button if the starter is professional and engaging, or **👎 (Thumbs Down)** if it feels generic or off-topic.
3. Select a rating from **1 to 5 Stars** using the interactive star widget.
4. Your feedback is instantly submitted to the backend and recorded in the analytics database.

### Step 4: Verify Technical Claims
1. Switch to the **"Fact-Check"** tab in the top navigation bar.
2. In the verification input box, type a technical claim, company name, or scientific concept you want to verify before your meeting.
   *Example input: "Transformer neural network architecture"*
3. Click **"Verify Fact"**.
4. Review the verification card, which displays the canonical summary from Wikipedia along with the system's **Confidence Score (`high`, `medium`, or `low`)**.
5. If the result returns a low confidence or fallback warning, use alternative wording or broader keywords.

### Step 5: Review History & Analytics
1. Click on the **"History"** tab to review all previously generated starters and extracted themes.
2. Use the search box to locate specific notes from past conferences.
3. Switch to the **"Analytics Dashboard"** tab to view bar charts summarizing your preparation volume and feedback ratings over time.

---

## 6. Frequently Asked Questions (FAQs)

#### Q1: Why is there no user login or account registration screen?
**A:** The Personalized Networking Assistant is architected as a local, privacy-focused Minimum Viable Product (MVP) for academic and competition evaluation. All session states and historical data are stored locally on your machine via atomic JSON files (`data/history.json` and `data/feedback.json`), eliminating the need for cloud databases or password management.

#### Q2: How does the AI model generate starters without internet access?
**A:** The application utilizes Hugging Face's `facebook/bart-large-mnli` and `gpt2` (124M parameter) models running locally on your CPU. Once downloaded during the initial setup or Docker build, inference happens entirely offline inside your local environment. Internet access is only required when using the Fact-Checker module to query the live Wikipedia REST API.

#### Q3: What is "Zero-Shot Classification" and why is BART used?
**A:** Zero-shot classification allows an AI model to categorize text into classes or themes it hasn't explicitly been trained on. We use `facebook/bart-large-mnli` because it has been pre-trained on Natural Language Inference (NLI) tasks, enabling it to accurately inspect an event description and determine relevance across arbitrary professional themes without requiring custom fine-tuning.

#### Q4: Where is my historical data and feedback saved? Can I export or delete it?
**A:** Your data is stored locally in the `data/` directory located in the project root:
- History: `data/history.json`
- Feedback: `data/feedback.json`
You can back up, export, or clear your history simply by copying or deleting these JSON files. The backend uses atomic file operations to ensure data integrity.

#### Q5: Can I connect my own OpenAI, Anthropic, or Hugging Face API keys?
**A:** The current competition architecture is designed to run 100% locally using open-source models (`BART` and `GPT-2`) to demonstrate self-contained edge/CPU inference without commercial API dependencies or costs. However, the backend architecture is modular, allowing developers to extend `app/services/` with external API providers if desired.

#### Q6: How does the Fact-Checker determine confidence scores (`high`, `medium`, `low`)?
**A:** The confidence scoring engine evaluates multiple algorithmic factors upon receiving a response from the Wikipedia REST API:
- **High:** Exact title match, HTTP 200 status, and substantial content length indicating a well-documented canonical topic.
- **Medium:** Partial keyword match, disambiguation page detected, or minor lexical divergence from the search query.
- **Low:** Missing entity, HTTP 404/fallback triggered, or sparse summary data requiring manual verification.

#### Q7: Why is the very first request to generate starters slightly slower than subsequent requests?
**A:** We implement a **Lazy-Loaded Singleton Pattern** in the backend (`app/services/`). To conserve system RAM and CPU cycles during startup, the heavy ML models (`BART` and `GPT-2`) are not loaded into memory when FastAPI starts. Instead, they are loaded into memory upon the *first* API request. All subsequent requests reuse the in-memory singleton instances, resulting in rapid inference times.

#### Q8: What happens if Wikipedia is unreachable or offline during a fact-check?
**A:** The system incorporates a **Defensive Fallback Mechanism**. If an HTTP timeout, DNS failure, or rate-limit occurs while querying the Wikipedia REST API, the backend catches the exception and returns a structured fallback response with a `low` confidence score and an explanation message. The Streamlit UI displays this gracefully without crashing or losing user state.

---

## 7. User Troubleshooting Guide

If you encounter issues while using the Personalized Networking Assistant, consult the table below for immediate diagnostic steps and solutions.

| Symptom / Error Message | Probable Root Cause | Step-by-Step Solution |
| :--- | :--- | :--- |
| **"Error connecting to backend API at `http://localhost:8000`"** | The FastAPI backend server is not running or has crashed. | **1.** Check your terminal or Docker logs.<br>**2.** If running locally, ensure you activated your virtualenv and ran `uvicorn app.main:app --port 8000`.<br>**3.** If using Docker, run `docker-compose up -d --build` and verify containers with `docker ps`. |
| **"Port 8000 (or 8501) is already in use"** | Another local process or background Docker container is occupying the port. | **1.** On Windows PowerShell: run `netstat -ano \| findstr :8000` to find the Process ID (PID).<br>**2.** Terminate the conflicting process: `taskkill /PID <PID> /F`.<br>**3.** Alternatively, restart Docker Desktop or run `docker-compose down`. |
| **First request spins for 30+ seconds without generating text** | The Lazy-Loaded AI models (`BART`/`GPT-2`) are being loaded into memory or downloaded for the first time. | **1.** Be patient on the initial request; model loading on CPU can take 15–45 seconds depending on hardware.<br>**2.** Ensure your machine has at least 4GB of available RAM.<br>**3.** Verify in terminal logs: look for `Loading BART model...` and `Model loaded successfully`. |
| **Fact-Checker returns "Fallback Mode: Unable to verify claim at this time"** | No internet connection, firewall blocking outbound HTTP requests, or Wikipedia API rate-limiting. | **1.** Verify your machine has active internet connectivity.<br>**2.** Ensure no corporate VPN or firewall is blocking outbound traffic to `https://en.wikipedia.org`.<br>**3.** Try searching for a simpler, canonical keyword (e.g., search `"Artificial intelligence"` instead of a full sentence). |
| **"Permission denied" or "JSONDecodeError" when saving feedback/history** | The JSON files in `data/` were manually edited with invalid syntax or have read-only file permissions. | **1.** Navigate to the `data/` directory.<br>**2.** Open `history.json` and `feedback.json` in a text editor and ensure they contain valid JSON arrays (`[]`).<br>**3.** If corrupted, delete the JSON files; the backend will automatically regenerate clean atomic files on the next request. |
| **Streamlit UI appears frozen or layout is misaligned** | Browser cache conflict or Streamlit websocket disconnection. | **1.** Hard-refresh your web browser (`Ctrl + F5` or `Cmd + Shift + R`).<br>**2.** Click the **"Rerun"** or **"Clear Cache"** options in the top-right Streamlit hamburger menu.<br>**3.** Restart the Streamlit frontend service if necessary. |
