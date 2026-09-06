# ParsAIce: Master Project Plan

## Pre-Phase 1: Foundation & Core Backend Architecture

### 1. Project Overview
**ParsAIce** is an AI-powered automated landing page generator. It utilizes a multi-agent architecture to take a user's business idea and automatically generate, critique, and build a complete landing page structure, saving the results to a database for frontend rendering.

### 2. Pre-Phase 1 Objectives
The goal of Pre-Phase 1 is to establish a fully functional, headless backend capable of receiving API requests, queuing background tasks, orchestrating AI agents, and persisting data. 
* **Status:** ✅ COMPLETED

### 3. Technology Stack
* **Backend Framework:** FastAPI (Python)
* **Task Queue:** Celery
* **Message Broker:** Redis (Hosted via Upstash)
* **Database:** Supabase (PostgreSQL)
* **AI Providers:** 
  * Primary: Anthropic (Claude Sonnet 5)
  * Fallback: OpenAI (GPT-4o)
* **Environment:** Windows 11 / PowerShell

### 4. System Architecture & Data Flow
1. **Ingestion:** User sends a POST request to `/api/projects` via FastAPI.
2. **Persistence:** Project metadata is saved to Supabase with a status of `generating`.
3. **Queueing:** FastAPI dispatches a `generate_landing_page_task` to the Celery worker via Redis.
4. **Orchestration (The AI Pipeline):**
   * **Agent 1 (Planner):** Generates the initial structure and copy.
   * **Agent 2 (Critic):** Reviews the plan for quality and feasibility.
   * **Agent 3 (Builder):** Generates the final code/content based on the critique.
5. **Completion:** The final output is saved back to Supabase, and the task status is updated to `success`.

### 5. Critical Pre-Phase 1 Milestones & Resolutions
This section documents the core infrastructure hurdles overcome during setup.

#### 5.1. Environment & Dependencies
* **Milestone:** Virtual environment creation and dependency installation.
* **Resolution:** Installed `fastapi`, `uvicorn`, `celery`, `redis`, `supabase`, `anthropic`, and `openai`.

#### 5.2. Windows Celery Compatibility
* **Issue:** Celery does not natively support Windows multiprocessing.
* **Resolution:** Configured the Celery worker to run with the `--pool=solo` flag to prevent event loop crashes.

#### 5.3. AI API Integration & Model Deprecation
* **Issue:** Anthropic API returned `404 Not Found` errors due to using deprecated model IDs (e.g., `claude-3-haiku-20240307`).
* **Resolution:** Updated the AI Router to use the current, active model ID: `claude-sonnet-5`.

#### 5.4. Anthropic "Extended Thinking" Response Parsing
* **Issue:** The new `claude-sonnet-5` model returns a `ThinkingBlock` before the actual text, causing an `AttributeError` when the code tried to read `.text` from the first block.
* **Resolution:** Refactored `backend/ai_router.py` to iterate through the response blocks and extract only those where `block.type == "text"`.

### 6. Verified End-to-End Workflow
The following sequence was successfully executed and logged in the Celery worker:
1. `Task backend.worker.generate_landing_page_task received`
2. `Sending request to Anthropic (Sonnet 5)...`
3. `Planner finished.` (HTTP 200 OK)
4. `Critic finished.` (HTTP 200 OK)
5. `Builder finished.` (HTTP 200 OK)
6. `Project saved to database!`
7. `Task succeeded in 95.85s: 'Success'`

### 7. Next Steps: Transition to Phase 1
With the backend fully operational, Phase 1 will focus on the user-facing application and API expansion.

* [ ] **Frontend Initialization:** Set up React/Next.js or preferred frontend framework.
* [ ] **UI Development:** Build the project submission form and dashboard.
* [ ] **Real-time Status Polling:** Implement frontend polling or WebSockets to show the "Planner -> Critic -> Builder" progress to the user.
* [ ] **Landing Page Renderer:** Build the component that takes the generated JSON/HTML from the database and renders the actual landing page.
* [ ] **Error Handling:** Implement robust retry logic and user-facing error messages for AI failures.

---
*Document generated upon successful completion of Pre-Phase 1 backend integration.*