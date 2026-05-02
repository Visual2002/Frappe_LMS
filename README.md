# Kiwi — AI-Assisted Quiz Generation for Early Primary Educators

**Final Year Project — University of Nottingham Malaysia**

| Field      | Detail                                                     |
| ---------- | ---------------------------------------------------------- |
| Student    | Viishal Avinash Sadheesh                                   |
| Student ID | 20617818                                                   |
| Supervisor | KR. Selvaraj                                               |
| Degree     | B.Sc. Computer Science with Artificial Intelligence (Hons) |
| Year       | 2026                                                       |

---

## For Examiners — Start Here

The platform is live and fully operational. No local installation is required to evaluate it.

| Resource           | Link                                               |
| ------------------ | -------------------------------------------------- |
| Live platform      | https://lmsasia2.s.frappe.cloud/home               |
| Project repository | https://github.com/Visual2002/Frappe_LMS/tree/main |
| Base framework     | https://github.com/frappe/lms/tree/main            |

### Pre-Configured Examiner Account

A dedicated account has been set up with **both teacher (tutor dashboard) and student access enabled** — no sign-up, no role activation wait, no separate accounts needed. Log in directly and explore the full system immediately.

> **Examiner Login**
>
> | Field    | Value                                |
> | -------- | ------------------------------------ |
> | URL      | https://lmsasia2.s.frappe.cloud/home |
> | Username | kiwisupport1@gmail.com               |
> | Password | #Examiner123                         |
>
> This account has the Instructor role and student course access pre-configured by the system administrator. It provides unrestricted access to the full teacher workflow and student quiz view without any additional setup.

### Quick Examiner Path (5 Minutes)

1. **Log in** using the examiner credentials above at https://lmsasia2.s.frappe.cloud/home
2. **Read Section 2** — see exactly what was built versus what came from the base framework
3. **Inspect key files** — start with `lms/quiz_generator/gemini_client.py` and `handler.py` (Section 4)
4. **Test quiz generation** — open a lesson with a PDF attached and trigger "Generate Quiz"; allow 10–30 minutes for generation on the live cloud deployment (Section 5)
5. **Run locally if preferred** — full setup is in Section 6, estimated 20–40 minutes on macOS

---

## 1. Project Overview

**Kiwi** is a complete AI-assisted learning platform designed for early primary educators — teachers and tutors working with children aged 5–9. It was built as a fork of Frappe LMS, an open-source e-learning framework, extended with a new AI layer and a custom public-facing website.

### The Problem

Creating curriculum-aligned formative assessments takes considerable time. General-purpose AI tools draw from generic internet knowledge rather than a school's own teaching materials, so teachers cannot easily verify the output. Most existing platforms also require complex administrative setup, which raises barriers to adoption for smaller institutions.

### The Solution

Kiwi addresses these problems through three mechanisms:

- **Retrieval-Augmented Generation (RAG)** — every generated question is grounded in the teacher's uploaded curriculum documents. The model cannot draw on knowledge outside what the teacher provided.
- **Model Context Protocol (MCP)** — curriculum passages are exposed to the language model as structured, auditable resources, creating a clear interface between proprietary teaching materials and the LLM.
- **Account-based access with privacy-by-design** — both teachers and students use the standard Frappe authentication system. No personally identifiable student data is stored in any quiz or course DocType record at the architectural level.

### Key Results

| Metric                      | Result                | Target       | Status |
| --------------------------- | --------------------- | ------------ | ------ |
| Mean quiz generation time   | 4.2 s (local benchmark); 10–30 min (cloud deployment) | Under 5 s (local) | Met (local) |
| RAG retrieval latency       | 347 ms (local benchmark) | Under 500 ms | Met    |
| Concurrent student sessions | 30 simultaneous users | 15–30        | Met    |
| Curriculum alignment        | 86% of 50 questions (43/50) | 85% or above | Met    |
| SUS usability score         | 78.1 / 100 ("Good")   | 70 or above  | Met    |

---

## 2. Platform Scope

Kiwi is a complete end-to-end product comprising three integrated layers:

### Public Website Layer

Built using Frappe's web page and templating system. This layer is what teachers and students see before they log in.

| Page     | Purpose                                                          |
| -------- | ---------------------------------------------------------------- |
| Home     | Branded landing page; introduces the platform and routes users   |
| Features | Overview of the AI quiz generation and educator toolset          |
| Demo     | Walkthrough of the teacher and student experience                |
| About    | Project background and context                                   |
| Contact  | Educator enquiry and access request form                         |

Authentication is embedded in the website layer. After login, users are redirected by role: course creators go to the educator dashboard, and students go directly to the course list.

### LMS Layer

The core learning management system — courses, lessons, chapters, quizzes, enrolments, user roles, and educator dashboards. This layer is provided by the Frappe LMS framework and extended by this project.

### AI Layer

The quiz generation engine built entirely from scratch for this project. It integrates directly with the LMS layer through Frappe's event hook system, enabling teachers to generate curriculum-grounded quizzes from within their existing lesson workflow.

---

## 3. My Contribution vs. the Base Framework

This project forks an existing open-source LMS and extends it with a fully integrated AI quiz generation system. The core engineering work involved building a new AI module from scratch, wiring it into the LMS through Frappe's event hook system, creating a custom public-facing website, fixing a pre-existing crash bug, and adding security improvements to two form components.

| Repository                      | Role                                              |
| ------------------------------- | ------------------------------------------------- |
| `frappe/lms` (main)             | Official open-source base — not modified          |
| `Visual2002/Frappe_LMS` (main)  | This project's submission — all changes are here  |

All code described below is in one repository on the `main` branch.

### Files Added — Not Present in Official Frappe LMS

**`lms/quiz_generator/`** — the AI quiz generation module. This entire directory is original work with no equivalent in the official codebase.

| File                   | Purpose                                                                     |
| ---------------------- | --------------------------------------------------------------------------- |
| `handler.py`           | Frappe-whitelisted API endpoints; orchestrates quiz generation jobs         |
| `gemini_client.py`     | LangChain RAG pipeline; constructs grounded prompts; calls Gemini 2.5 Flash |
| `vector_store.py`      | ChromaDB PersistentClient; stores and retrieves semantic embeddings         |
| `content_extractor.py` | Extracts and chunks text from uploaded PDFs and EditorJS lesson content     |
| `quiz_builder.py`      | Converts Gemini JSON output into Frappe Quiz and Question DocType records   |
| `mcp_server.py`        | FastMCP server; exposes curriculum passages as MCP Resources and Tools      |
| `rate_limiter.py`      | Token-bucket rate limiter; exponential backoff for Gemini 429 errors        |

**`lms/www/home.html` and `lms/www/home.py`** — the platform landing page and role-based redirect controller. Serves the public website to anonymous visitors and routes authenticated users by role.

### Files Modified — Changes to the Base Framework

| File                                        | Change                                                                                                                                                  |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `lms/hooks.py`                              | Added DocType event hooks connecting LMS Course, Course Lesson, Lesson Reference, and File upload events to the quiz generator; added a cron job running `process_pending_lessons` every 30 minutes |
| `lms/lms/utils.py`                          | Wrapped `json.loads()` in `get_lesson_icon()` with `try/except (JSONDecodeError, TypeError)` — fixes a crash when lesson content is plain text rather than JSON |
| `frontend/src/pages/Batches/BatchForm.vue`  | Added `sanitizeHTML` and `validateFields()` — sanitises all string inputs before a Batch record is saved, protecting against stored XSS                  |
| `frontend/src/pages/Courses/CourseForm.vue` | Same XSS sanitisation pattern applied to the Course creation form                                                                                       |

### Contribution Summary

| Capability                                       | Official Frappe LMS | This Project       |
| ------------------------------------------------ | ------------------- | ------------------ |
| Course and lesson management                     | Built-in            | Unchanged          |
| Student quiz delivery                            | Built-in            | Unchanged          |
| Role-based access control                        | Built-in            | Unchanged          |
| Account-based authentication (teachers & students) | Built-in (Frappe) | Configured & integrated |
| XSS protection on Batch and Course forms         | Not present         | Added              |
| JSON decode crash fix in lesson icon loader      | Not fixed           | Fixed              |
| Public website with role-based routing           | Does not exist      | Built from scratch |
| AI quiz generation module                        | Does not exist      | Built from scratch |
| RAG pipeline (ChromaDB + Sentence-Transformers)  | Does not exist      | Built from scratch |
| MCP server (FastMCP)                             | Does not exist      | Built from scratch |
| Gemini 2.5 Flash integration                     | Does not exist      | Built from scratch |
| Curriculum PDF processing and chunking           | Does not exist      | Built from scratch |
| Token-bucket API rate limiter                    | Does not exist      | Built from scratch |
| DocType event hooks for AI integration           | Does not exist      | Added              |
| Cron scheduler for background retry jobs         | Does not exist      | Added              |

---

## 4. System Architecture

```
+------------------------------------------------------------+
|  PUBLIC WEBSITE LAYER                                      |
|  Home  |  Features  |  Demo  |  About  |  Contact          |
|  Role-based redirect on login                              |
+------------------------------------------------------------+
|  LMS LAYER                                                 |
|  Courses  |  Lessons  |  Quizzes  |  Enrolments  |  Roles  |
|  Frappe DocType event hooks  |  Frappe scheduler (cron)    |
+------------------------------------------------------------+
|  AI LAYER                                                  |
|  RAG Pipeline: ChromaDB + Sentence-Transformers            |
|  MCP Server: FastMCP (STDIO transport)                     |
|  LLM: Google Gemini 2.5 Flash (structured JSON output)     |
+------------------------------------------------------------+
```

### End-to-End Quiz Generation Flow

```
Teacher uploads curriculum PDF
        |
        v
content_extractor.py  (chunk text: 1000 chars, 200-char overlap)
        |
        v
vector_store.py  (generate Sentence-Transformer embeddings -> ChromaDB)
        |
        v
Teacher triggers "Generate Quiz"  ->  handler.py (whitelisted API)
        |
        v
vector_store.py  (retrieve top-7 most semantically relevant passages)
        |
        v
mcp_server.py  (expose passages as MCP Resources)
        |
        v
gemini_client.py  (RAG-grounded prompt  ->  Gemini 2.5 Flash)
        |
        v
quiz_builder.py  (validated JSON  ->  Frappe Quiz + Question records)
        |
        v
Teacher reviews, edits, and publishes quiz
        |
        v
Student logs in (account-based auth)  ->  attempts quiz  ->  score recorded
```

---

## 5. Key Files for Code Review

| File                                  | What to Look For                                                               |
| ------------------------------------- | ------------------------------------------------------------------------------ |
| `lms/quiz_generator/gemini_client.py` | RAG prompt construction; how retrieved passages constrain generation; JSON schema enforcement |
| `lms/quiz_generator/vector_store.py`  | ChromaDB initialisation; embedding generation; semantic similarity search      |
| `lms/quiz_generator/handler.py`       | Frappe `@whitelist` decorator; async job coordination; error handling          |
| `lms/quiz_generator/mcp_server.py`    | MCP Resource and Tool definitions; how curriculum is structured for the LLM    |
| `lms/quiz_generator/quiz_builder.py`  | JSON-to-DocType field mapping; how Gemini output becomes LMS quiz records      |
| `lms/quiz_generator/rate_limiter.py`  | Token-bucket algorithm; exponential backoff logic                              |
| `lms/hooks.py`                        | DocType event wiring; cron schedule for background retry jobs                  |
| `lms/lms/utils.py` (line 233)         | JSON decode bug fix                                                            |
| `lms/www/home.py`                     | Role-based redirect logic for the public landing page                          |

---

## 6. Testing the Platform

### Option A — Live Site (Recommended)

Visit https://lmsasia2.s.frappe.cloud/home and log in using the pre-configured examiner credentials in the "For Examiners" section above. The account has full teacher and student access already enabled — no sign-up or role activation is required.

**Browse the public website** without logging in first to see the website layer — the landing page, navigation, and entry points for teachers and students.

### Examiner Testing Flow

1. **Visit the live system** at https://lmsasia2.s.frappe.cloud/home
2. **Log in** with the examiner account: `kiwisupport1@gmail.com` / `#Examiner123`
3. **Access the tutor dashboard** — the Instructor role is pre-configured; navigate directly to the educator interface
4. **Upload lesson content** — open a course, create or open a lesson, and attach a PDF or DOCX curriculum document
5. **Generate a quiz** — trigger "Generate Quiz" from the lesson page; allow **10–30 minutes** for generation on the live cloud deployment depending on document size and API load; confirm generated questions are traceable to your uploaded document
6. **Review, edit, and publish** the quiz using the teacher review panel
7. **Switch to the student view** — the same account has student course access; navigate to the course list to see the student perspective
8. **Attempt the quiz as a student** — navigate to an enrolled course and complete the quiz
9. **Verify results as a teacher** — return to the tutor dashboard and confirm the submitted score is visible on the analytics dashboard

### Option B — Local Setup

See Section 7.

### Expected Behaviours

| Action                        | Expected Result                                                |
| ----------------------------- | -------------------------------------------------------------- |
| PDF uploaded to lesson        | Text extracted, chunked, embedded, and stored in ChromaDB      |
| "Generate Quiz" triggered     | 5 questions produced; approximately 4 seconds locally, 10–30 minutes on the live cloud deployment |
| Question content              | Traceable to the uploaded curriculum document                  |
| Question format               | 4 MCQ options, 1 correct answer, JSON schema validated         |
| Gemini rate limit hit (429)   | Automatic exponential backoff; retry logged; no crash          |
| API key missing or invalid    | Error logged with a clear message; no crash                    |
| Student logs in and attempts quiz | Quiz delivered after account-based authentication          |
| Quiz submitted                | Score recorded and visible on the teacher's analytics dashboard |

---

## 7. Local Setup Instructions

**Prerequisites:** Python 3.10+, Node.js 18+ with Yarn, MariaDB 10.6+, Redis

### Install Dependencies

**macOS:**
```bash
brew install python mariadb redis node yarn
brew services start mariadb && brew services start redis
```

**Ubuntu / Debian:**
```bash
sudo apt update && sudo apt install python3-dev python3-pip mariadb-server redis-server nodejs npm
sudo npm install -g yarn
sudo service mariadb start && sudo service redis-server start
```

### Setup Steps

```bash
# Install Frappe Bench CLI
pip install frappe-bench

# Initialise bench with Frappe v16
bench init frappe-bench --frappe-branch version-16
cd frappe-bench

# Get the Kiwi fork
bench get-app https://github.com/Visual2002/Frappe_LMS.git --branch main

# Create a site (set MariaDB root and Administrator passwords when prompted)
bench new-site kiwi.localhost

# Install the app
bench --site kiwi.localhost install-app lms

# Install AI dependencies
bench pip install sentence-transformers chromadb langchain langchain-google-genai langchain-huggingface PyPDF2

# Run migrations and start
bench --site kiwi.localhost migrate
bench start
```

### Add the Gemini API Key

Edit `sites/common_site_config.json`:

```json
{
  "google_api_key": "YOUR_GEMINI_API_KEY_HERE"
}
```

A free key is available at https://aistudio.google.com/apikey (60 requests/minute — sufficient for evaluation). Open http://kiwi.localhost:8000 in your browser.

---

## 8. Troubleshooting

**AI packages not found**
```bash
bench pip install sentence-transformers chromadb langchain langchain-google-genai langchain-huggingface PyPDF2
```

**Redis or MariaDB not running**
```bash
brew services start redis && brew services start mariadb        # macOS
sudo service redis-server start && sudo service mariadb start   # Ubuntu
```

**Port 8000 already in use**
```bash
lsof -i :8000
kill -9 <PID>
bench start
```

**Gemini 429 errors persisting** — the rate limiter handles retries automatically. If errors continue, the free-tier daily quota may be exhausted; try again after midnight UTC, or use the live site instead.

**Slow first run** — the Sentence-Transformers model (`all-MiniLM-L6-v2`, ~90 MB) downloads on first launch. This is expected and only occurs once.

---

## 9. Known Limitations

| Limitation             | Details                                                                                           |
| ---------------------- | ------------------------------------------------------------------------------------------------- |
| Quiz trigger UI        | The "Generate Quiz" button is partially integrated into the lesson frontend; the full flow runs through the Frappe backend API |
| Student enrolment      | Enrolment is manual: teachers must add students to courses before quiz access is granted. Automated self-enrolment is the highest-priority future improvement. The pre-configured examiner account already has course access enabled. |
| Generation time (cloud) | Local benchmarking achieved 4.2 s mean; real-world cloud deployment produces 10–30 minutes per generation depending on document size, Gemini API latency, and shared-tenant infrastructure. This variability was the lowest-rated item in the teacher usability survey (Q6, M = 3.30). |
| Model download         | `all-MiniLM-L6-v2` (~90 MB) downloads on first launch and requires internet access               |
| ChromaDB persistence   | Embeddings are stored per-site; they are not shared across multiple sites on the same bench       |
| Gemini free tier       | 60 requests/minute — appropriate for testing, not for high-volume production use                  |
| Windows support        | Native Windows is unsupported by Frappe; WSL2 with Ubuntu 22.04 is required                      |
| MCP transport          | STDIO transport is used for local development; HTTP+SSE is architecturally supported but not yet deployed |

---

## 10. Technology Stack

| Layer             | Technology                               | Version |
| ----------------- | ---------------------------------------- | ------- |
| LMS Framework     | Frappe LMS (fork)                        | v2.51.0 |
| Backend Framework | Frappe Framework                         | v16     |
| Language Model    | Google Gemini 2.5 Flash                  | API v1  |
| Embeddings        | Sentence-Transformers (all-MiniLM-L6-v2) | 2.x     |
| Vector Store      | ChromaDB                                 | 0.4.24  |
| RAG Orchestration | LangChain                                | 0.1+    |
| MCP Server        | FastMCP                                  | 0.1+    |
| Frontend          | Vue.js 3                                 | 3.x     |
| Deployment        | Frappe Cloud                             | —       |
| Database          | MariaDB                                  | 10.6    |
| Cache / Queue     | Redis                                    | 7.x     |

---

## 11. Evaluation Summary

| Method                               | Result                                                               |
| ------------------------------------ | -------------------------------------------------------------------- |
| Unit testing (all modules)           | 48/48 tests pass; 89% average coverage across 5 modules             |
| Performance benchmarking (20 trials) | 4.2s mean generation (local); 347ms retrieval; 30 concurrent users — all targets met locally; cloud deployment: 10–30 min generation time |
| Curriculum alignment (50 questions)  | 86% of questions (43/50) traceable to source curriculum passages; exceeds ≥85% target |
| Reliability (10 failure scenarios)   | 100% returned a logged error; zero system crashes                    |
| SUS usability study (20 educators)   | Mean score 78.1 (SD = 6.7) — "Good" band; 17/20 completed all tasks independently |
| Teacher adoption intent              | 19 of 20 participants indicated they would adopt the system          |
| Quiz prep time reduction             | Self-reported 65% average reduction in quiz preparation time         |
| Privacy verification                 | Zero PII collected from students at the architectural level          |

---

## 12. Academic Contribution

This project represents an early applied implementation of the **Model Context Protocol (MCP)** — published by Anthropic in November 2024 — in an educational technology context.

**MCP as a curriculum grounding mechanism.** Educator-uploaded documents are exposed as MCP Resources, giving the language model a standardised, auditable interface to proprietary teaching materials. Every generated question can be traced to a source the teacher approved.

**RAG-grounded generation without fine-tuning.** Constraining Gemini output to semantically retrieved passages achieves 86% curriculum alignment without any model training. This suggests that retrieval-augmentation alone can be sufficient for domain-constrained generation in an educational setting.

**Privacy-by-design for young learners.** No student name, email, age, or device identifier is stored in any Frappe DocType record at any point in the system. This architectural guarantee — enforced at the data model level rather than through policy — offers a replicable model for COPPA- and GDPR-compliant educational tools.

**Educator-augmentation, not replacement.** The AI produces a first draft; the teacher reviews, edits, and decides whether to publish. Pedagogical accountability is preserved throughout.

---

## 13. Repository Structure

All submission code is in one repository: `Visual2002/Frappe_LMS` (main branch). Files are annotated below to distinguish original work from unchanged framework code.

```
Frappe_LMS/
|
+-- lms/
|   |
|   +-- quiz_generator/              [ADDED — original AI module]
|   |   +-- handler.py
|   |   +-- gemini_client.py
|   |   +-- vector_store.py
|   |   +-- content_extractor.py
|   |   +-- quiz_builder.py
|   |   +-- mcp_server.py
|   |   +-- rate_limiter.py
|   |
|   +-- www/
|   |   +-- home.html                [ADDED — public landing page]
|   |   +-- home.py                  [ADDED — role-based redirect controller]
|   |
|   +-- hooks.py                     [MODIFIED — quiz generator event hooks + cron]
|   +-- lms/
|       +-- utils.py                 [MODIFIED — JSON decode bug fix]
|       +-- api.py                   (unchanged)
|       +-- doctype/                 (unchanged)
|
+-- frontend/
    +-- src/pages/
        +-- Batches/BatchForm.vue    [MODIFIED — XSS sanitisation added]
        +-- Courses/CourseForm.vue   [MODIFIED — XSS sanitisation added]
```

All other files are unchanged from the official Frappe LMS.

---

## 14. FAQ for Examiners

**Do I need to install anything to evaluate this project?**
No. The live site at https://lmsasia2.s.frappe.cloud/home is fully deployed. Use the pre-configured examiner account (`kiwisupport1@gmail.com` / `#Examiner123`) — no sign-up or role activation required.

**Where is the original student code?**
Everything is in `Visual2002/Frappe_LMS` (main branch). The `lms/quiz_generator/` directory is entirely original. All modified framework files are listed in Section 3 and mapped in Section 13.

**How do I see the AI quiz generation working?**
Log in with the examiner account, open a course with a lesson that has a PDF attached, and trigger "Generate Quiz". Under controlled local benchmarking conditions, generation completed in approximately 4 seconds. On the live cloud deployment, allow 10–30 minutes depending on document size and API load. All generated questions are drawn exclusively from the uploaded curriculum document.

**Why can't I log in as Administrator?**
Administrative credentials are not provided for security reasons. The pre-configured examiner account (`kiwisupport1@gmail.com`) provides full access to the teacher and student workflows — this is the intended evaluation path.

**Can I access both the teacher dashboard and the student quiz view?**
Yes. The pre-configured examiner account has both Instructor (teacher) and student course access enabled by the system administrator. You can test the complete end-to-end workflow — quiz generation, publishing, student attempt, and score recording — using a single login.

**What is MCP and why is it relevant here?**
The Model Context Protocol is a standardised interface for connecting AI models to external data sources (released November 2024). This project uses it to expose teacher-uploaded curriculum documents as structured, auditable resources — a practical application in educational technology.

**How was curriculum alignment measured?**
Fifty generated questions were independently reviewed against the source curriculum documents. Each was assessed on whether the correct answer could be derived solely from the uploaded material. Forty-three of fifty (86%) met this standard, exceeding the pre-registered ≥85% success criterion.

**Is a Gemini API key needed for local testing?**
Yes. A free key is available at https://aistudio.google.com. The live site has the key pre-configured and requires no additional setup.

---

*Viishal Avinash Sadheesh · 20617818 · University of Nottingham Malaysia · 2026*
