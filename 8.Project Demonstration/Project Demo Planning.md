# Project Demo Planning: Personalized Networking Assistant

## 1. Executive Summary
A successful technical demonstration requires meticulous logistical planning, rigorous environment verification, precise time management, and flawless team coordination. This document establishes the master planning framework for the live presentation of the **Personalized Networking Assistant**. It details hardware and software prerequisites, pre-demo preparation timelines, an exact 15-minute minute-by-minute speaking schedule, role allocations, contingency protocols, and post-demo follow-up workflows.

---

## 2. Demo Environment Setup Checklist

To ensure deterministic execution during the live presentation, the demonstration host machine must be configured according to the following mandatory setup checklist:

### 2.1 Software & Environment Verification
* [x] **Operating System:** Windows 10/11 Pro (64-bit) or macOS Sonoma / Linux Ubuntu 22.04 LTS.
* [x] **Python Runtime:** Python 3.10.x or 3.11.x installed and added to system `PATH`.
* [x] **Virtual Environment:** Clean virtual environment (`venv` or `conda`) initialized and activated:
  ```powershell
  python -m venv venv
  .\venv\Scripts\activate
  pip install --upgrade pip
  pip install -r requirements.txt
  ```
* [x] **Hugging Face Model Cache:** Verify BART (`facebook/bart-large-mnli`) and GPT-2 (`gpt2`) model weights are pre-downloaded and verified in local disk cache (`~/.cache/huggingface/hub/`), ensuring zero network download latency during model initialization.
* [x] **Port Availability Check:** Verify that TCP ports **8000** (FastAPI Backend) and **8501** (Streamlit Frontend) are unassigned and open:
  ```powershell
  netstat -ano | findstr :8000
  netstat -ano | findstr :8501
  ```
* [x] **Containerization (Optional/Backup):** Verify Docker Desktop is running and Docker Compose builds cleanly:
  ```powershell
  docker-compose build
  docker-compose up -d
  ```
* [x] **Browser Configuration:** Google Chrome installed with all browser extensions (ad-blockers, script blockers) disabled in a clean Incognito/Private profile to prevent UI rendering interference.

---

## 3. Hardware Requirements for Demo

Running deep learning NLP models locally alongside a responsive web application requires robust system hardware. The team has established minimum and recommended hardware specifications for the demonstration machine:

| Hardware Component | Minimum Specification | Recommended Specification (Demo Host) | Purpose / Justification |
| :--- | :--- | :--- | :--- |
| **CPU** | 4 Cores / 8 Threads (Intel i5 10th Gen / AMD Ryzen 5) | **8+ Cores / 16 Threads** (Intel i7/i9 12th+ Gen / AMD Ryzen 7/9 or Apple M2/M3 Pro) | Rapid tokenization and sequence generation during local GPT-2 inference. |
| **RAM** | 16 GB DDR4 | **32 GB DDR4/DDR5** | Simultaneous loading of BART (~1.6GB) and GPT-2 (~500MB) weights in memory alongside OS and browser overhead. |
| **Storage** | 10 GB Free NVMe/SSD Space | **25+ GB Free NVMe SSD Space** | Rapid disk read speeds for loading model checkpoints into memory in <2 seconds. |
| **GPU (Optional)** | Integrated Graphics | **NVIDIA RTX 3060 / 4060 (6GB+ VRAM) with CUDA 11.8+** | Accelerates PyTorch tensor operations by 4x, reducing starter generation latency from ~2.5s to ~0.6s. |
| **Network / Internet**| Stable 10 Mbps Broadband | **25+ Mbps High-Speed Wi-Fi & Ethernet Backup** | Low-latency HTTP GET queries to the Wikipedia REST API during Fact-Checking demonstrations. |
| **Display / Projector**| 1080p (1920x1080) Resolution | **1080p / 2K Display (150% DPI scaling)** | Ensures Streamlit widgets, charts, and Swagger UI text are crisp and legible on projector screens. |

---

## 4. Pre-Demo Preparation Timeline

To prevent last-minute configuration panics, the team adheres to a strict countdown protocol beginning 24 hours prior to the demonstration scheduled start time.

```
+-----------------------------------------------------------------------------------+
|                           PRE-DEMO COUNTDOWN TIMELINE                             |
+-----------------------------------------------------------------------------------+
| T-24 HOURS (Day Before Demo)                                                      |
| • Run full automated regression test suite (pytest -v) -> Assert 100% pass rate   |
| • Perform dry-run presentation with countdown timer -> Adjust speaking paces      |
| • Verify Hugging Face offline cache on primary and secondary laptop computers    |
| • Charge all laptop batteries to 100% and pack display adapters (HDMI/Type-C)     |
+-----------------------------------------------------------------------------------+
| T-1 HOUR (60 Minutes Before Demo)                                                 |
| • Arrive at presentation venue / join virtual staging room                        |
| • Connect primary laptop to power adapter and wired Ethernet (if available)       |
| • Launch FastAPI backend (uvicorn app:app --port 8000) and verify /health endpoint|
| • Launch Streamlit frontend (streamlit run streamlit_app.py --server.port 8501)   |
| • Pre-populate 3 test sessions in history.json for rich dashboard chart rendering |
+-----------------------------------------------------------------------------------+
| T-15 MINUTES (Final Readiness Check)                                              |
| • Open Chrome Incognito window to http://localhost:8501 and http://localhost:8000 |
| • Close all non-essential background applications (Slack, WhatsApp, Spotify, IDEs)|
| • Mute system notifications (Windows Focus Assist / macOS Do Not Disturb ON)      |
| • Perform one quick test generation; verify response latency < 2 seconds          |
+-----------------------------------------------------------------------------------+
```

---

## 5. Demo Timeline (15 Minutes Total)

The presentation is structured into a precise 15-minute time box designed to balance conceptual framing, live software execution, code transparency, and stakeholder engagement.

| Time Window | Duration | Segment Title | Activities & Visuals |
| :---: | :---: | :--- | :--- |
| **00:00 - 02:00** | 2 mins | **Introduction & Problem Statement** | • Slide 1-2: Welcome, project overview.<br>• Problem: Professional networking anxiety, generic conversation starters, and information overload at technical conferences.<br>• Solution: AI-tailored assistant combining BART, GPT-2, and live fact-checking. |
| **02:00 - 05:00** | 3 mins | **System Architecture & AI Models** | • Slide 3-4: Architecture diagram walkthrough (Streamlit ↔ FastAPI ↔ Hugging Face ↔ Wikipedia API).<br>• Explain BART zero-shot classification for theme extraction.<br>• Explain GPT-2 prompt engineering and decoding strategies (top-k/top-p sampling). |
| **05:00 - 10:00** | 5 mins | **Live Feature Walkthrough** | • Live UI Demo: Walk through Steps 1 to 9 of Feature Demonstration script.<br>• Show event analysis, theme badges, starter cards, thumbs up/down feedback.<br>• Demonstrate Wikipedia fact-checking on 'blockchain in healthcare'.<br>• Show persistent History tab and interactive analytics Dashboard charts. |
| **10:00 - 12:00** | 2 mins | **Backend API & Code Walkthrough** | • Show Swagger UI at `http://localhost:8000/docs`; execute live `/generate` POST request.<br>• Code walkthrough: Briefly display clean modular architecture in VS Code (`nlp_service.py`, `app.py`, `test_event_analyzer.py`).<br>• Present automated QA metrics: 38 pytest cases passed, >85% code coverage. |
| **12:00 - 15:00** | 3 mins | **Q&A & Concluding Remarks** | • Open floor to SmartBridge mentors and evaluators.<br>• Answer technical questions using prepared Q&A reference.<br>• Conclude with brief statement on future scalability (Phase 2 & Phase 3 roadmap). |

---

## 6. Role During Demo

**mehboob ali mohammed (Full-Stack Developer & Project Lead)**
* **On-Stage Role:** Presents the entire demonstration, covering problem statement, system architecture, AI/ML engine deep-dive, live feature walkthrough, backend API execution, and moderates the Q&A session.
* **Backstage Role:** Manages the presentation timer; monitors system RAM and CPU utilization during live inference; stands by to explain model hyperparameter choices and architectural decisions during technical Q&A.
* **Technical Responsibilities:** Ensures all services are running; handles live code walkthrough; responds to all mentor and evaluator questions using prepared reference materials.
* **Backstage Role:** Acts as primary note-taker during Q&A, recording all mentor feedback, feature requests, and evaluation scores for post-demo analysis.

### 6.4 mehboob ali mohammed (Frontend & Deployment Engineer)
* **On-Stage Role:** Operates the live keyboard and mouse; drives the Streamlit web application demonstration, showcasing user flows, interactive widgets, and dashboard analytics smoothly.
* **Backstage Role:** Prepares secondary backup laptop; ensures video recording walkthrough (`NetAssist_Demo_2026.mp4`) is cued and ready for immediate playback if needed.

---

## 7. Backup Plans for Common Demo Failures

Despite thorough preparation, live software demonstrations can encounter unexpected anomalies. The team has established immediate, rehearsed protocols for five common failure scenarios:

| Potential Failure Scenario | Root Cause | Immediate Mitigation Protocol | Assigned Handler |
| :--- | :--- | :--- | :--- |
| **1. Model Loading Timeout / Latency Spike** | High background CPU load or OS paging during live GPT-2 inference causing >5s delay. | Presenter smoothly continues speaking: *"While our local model completes inference, let's look at how our asynchronous backend handles queueing..."* If latency exceeds 10s, switch to pre-generated History tab. | **mehboob ali mohammed** |
| **2. Port Conflict (8000 or 8501 Already in Use)** | Orphaned Python process from previous practice session holding TCP port bound. | Execute emergency Powershell script: `taskkill /F /IM python.exe` and relaunch servers using pre-scripted batch file `run_demo.bat`. | **mehboob ali mohammed** |
| **3. Wi-Fi / Internet Drop During Fact-Check** | Venue router disconnection or DNS lookup failure when querying Wikipedia API. | Fact-checking module automatically catches `ConnectionError` and fails over to local offline knowledge dictionary. Presenter highlights: *"Notice our offline resilience..."* | **mehboob ali mohammed** |
| **4. Streamlit UI Freeze / Browser Crash** | Memory leak in browser tab or accidental window closure during live interaction. | Instantly hit `Ctrl + Shift + T` to restore tab or open pre-loaded secondary tab. If server crashed, switch projector display to backup laptop within 15 seconds. | **mehboob ali mohammed** |
| **5. Projector Resolution / HDMI Scaling Issues** | Projector cutting off UI sidebar or displaying blurry text due to incorrect DPI. | Use Windows shortcut `Ctrl + Plus (+)` to scale browser zoom to 125%/150%. If display fails entirely, share presentation via Google Meet link to mentors' laptops. | **mehboob ali mohammed** |

---

## 8. Post-Demo Follow-Up Actions

The project lifecycle does not end when the demonstration concludes. To maximize learning and ensure proper project closure, the team will execute the following structured follow-up actions within 48 hours post-demo:

1. **Mentor Feedback Synthesis (T+2 Hours):**
   * **mehboob ali mohammed** compiles all notes taken during the Q&A session into a formal document titled `Mentor_Evaluation_Feedback_Report.md`.
   * The team convenes for a 30-minute retrospective to review mentor comments and categorize feedback into *Immediate Fixes* and *Future Roadmap Enhancements*.

2. **Codebase Archiving & Tagging (T+12 Hours):**
   * **mehboob ali mohammed** merges any final documentation tweaks into the `main` branch.
   * **mehboob ali mohammed** creates an official Git release tag `v1.0.0-final-demo` and pushes the release artifact to GitHub, ensuring a permanent, immutable snapshot of the evaluated codebase:
     ```powershell
     git tag -a v1.0.0-final-demo -m "Official release tag for SmartBridge Final Project Demonstration"
     git push origin v1.0.0-final-demo
     ```

3. **Artifact & Presentation Preservation (T+24 Hours):**
   * Upload presentation slides (`.pptx` and `.pdf`), demo video walkthrough (`.mp4`), and final project documentation suite to the shared Google Drive repository and GitHub `docs/` folder.

4. **Stakeholder Appreciation & Sign-Off (T+48 Hours):**
   * Send a formal thank-you email to SmartBridge mentors and program coordinators, attaching the final release notes, presentation slides, and repository link, formally closing out the academic project milestone.
