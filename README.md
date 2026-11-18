# AI Job Search — Multi-Agent System

**Final Project — Multi-Agent Systems (SDAIA Academy)**
---
Sarah Sultan Alotaibi
Wadha Ail Alhelal

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![LangChain](https://img.shields.io/badge/LangChain-Core%20%7C%20Groq-green.svg)](https://www.langchain.com/)
[![Google Colab](https://img.shields.io/badge/Google%20Colab-Ready-orange.svg)](https://colab.research.google.com/)

---

## 📋 Table of Contents
- [Project Overview](#project-overview)
- [Project Structure](#project-structure)
- [Architecture Diagram](#architecture-diagram)
- [Workflow Diagram & Explanation](#workflow-diagram--explanation)
- [Agents and Their Roles](#agents-and-their-roles)
- [Security & Guardrails](#security--guardrails)
- [Technologies & AI Tools Used](#technologies--ai-tools-used)
- [Installation and Setup Instructions](#installation-and-setup-instructions)
- [How to Run the Project](#how-to-run-the-project)
- [Future Improvements](#future-improvements)
- [SDAIA Academy GitHub Repository Link](#sdaia-academy-github-repository-link)

---

## 🎯 Project Overview

The **AI Job Search Multi-Agent System** is an intelligent, centralized multi-agent career matching pipeline built to automate resume screening, job discovery, candidate-role scoring, and personalized career advising. 

Given a candidate's CV (in raw text or PDF format), the system validates and sanitizes the input against security threats, extracts a structured candidate profile, queries an external job database, scores the most relevant positions, and uses a **Reflection / Self-Critique** loop between a **Matching Agent** and a **Critic Agent** to ensure that only realistic job recommendations are delivered. If no suitable match is found, the system automatically pivots to generate a constructive **Skill-Gap Report**.

### Key Requirements Covered
- **5 Specialized Agents + 1 Central Orchestrator** (`JobSearchManager`)
- **Multi-Branch Decision Gates** (Security validation, score threshold routing, and Critic reflection approval)
- **Advanced Reasoning Pattern:** Reflection / Self-Critique loop (`MatchingAgent` ↔ `CriticAgent`)
- **External Tools:** Live Google Sheets CSV job database integration via `pandas` and PDF extraction via `pypdf`
- **Security & Guardrails:** Input validation, prompt injection detection, and PII masking

---

## 📁 Project Structure

```text
ai-job-search-multi-agent/
│
├── job_search_multi_agent.ipynb   # Main Google Colab notebook containing all steps, agents, and tests
├── jobs_demo.csv                  # Local fallback dataset of job listings (if Google Sheet is offline)
└── README.md                      # Project documentation
```

### Notebook Organization
| Step | Section | Description |
| :--- | :--- | :--- |
| **Step 1–4** | Setup & Configuration | Installs dependencies, loads `GROQ_API_KEY`, initializes `ChatGroq`, and sets thresholds (`MIN_SCORE = 60`, `MAX_REVISIONS = 2`, `TOP_N = 8`). |
| **Step 5–6** | Helpers & External Tools | Defines JSON parsing, whole-word skill matching, PDF reader (`read_cv_pdf`), and `JobDatabaseTool`. |
| **Step 6.5** | Security & Guardrails | Implements `SecurityGuardrails` for input validation, prompt injection detection, and PII masking. |
| **Step 7–11** | Specialized Agents | Defines `CVParserAgent`, `JobSearchAgent`, `MatchingAgent`, `CriticAgent`, and `RecommendationAgent`. |
| **Step 12** | Orchestrator | Implements `JobSearchManager` to coordinate agents, reflection loops, and decision gates. |
| **Step 13–16** | Utilities, Demos & Execution | Provides test CVs (`CV_STRONG`, `CV_JUNIOR`, `CV_WEAK`, `CV_MALICIOUS`), ranking tables, and report export. |

---

## 🏗️ Architecture Diagram

The system uses a **Centralized Multi-Agent Architecture** orchestrated by the `JobSearchManager`, guarded at entry by `SecurityGuardrails`, and connected to external data tools.

```mermaid
graph TB
    User([Candidate CV: PDF / Text]) --> Manager[Job Search Manager<br/>Central Orchestrator]

    subgraph Guardrails [Security & Guardrails Layer]
        Sec1[Input Validation &<br/>Prompt Injection Scanner]
        Sec2[PII Masking Engine<br/>Email / Phone / ID Redaction]
    end

    subgraph Agents [Specialized Multi-Agent Pipeline]
        A1[1. CV Parser Agent]
        A2[2. Job Search Agent]
        A3[3. Matching Agent]
        A4[4. Critic Agent<br/>Reflection / Self-Critique]
        A5[5. Recommendation Agent]
    end

    subgraph Tools [External Tools & Storage]
        PDF[pypdf Reader]
        DB[(Google Sheet Job Database<br/>via JobDatabaseTool)]
        OutFile[(job_recommendation.txt)]
    end

    User --> PDF --> Manager
    Manager <--> Guardrails
    Manager <--> A1
    Manager <--> A2
    A2 <--> DB
    Manager <--> A3
    Manager <--> A4
    Manager <--> A5
    A5 --> OutFile
```

---

## 🔄 Workflow Diagram & Explanation

```mermaid
flowchart TD
    A[CV Input: PDF or Text] --> B{Security Guardrails<br/>Validation & Injection Check}
    B -- Blocked --> C[Stop Pipeline & Return<br/>User-Friendly Security Notice]
    B -- Safe --> D[PII Masking<br/>Redact Email, Phone & ID]
    D --> E[1. CV Parser Agent<br/>Extract Structured JSON Profile]
    E --> F[2. Job Search Agent]
    DB[(Google Sheet Job Database)] <--> F
    F --> G[Top 8 Shortlisted Jobs]
    G --> H[3. Matching Agent<br/>Score Jobs 0–100 & Explain Fit]
    H --> I{Decision 1:<br/>Best Score >= 60?}
    I -- No --> J[5. Recommendation Agent<br/>Mode: Skill-Gap Report]
    I -- Yes --> K[4. Critic Agent<br/>Strict Senior Recruiter Review]
    K --> L{Decision 2:<br/>Approved by Critic?}
    L -- "No (Max 2 Revisions)" --> H
    L -- Yes --> M[5. Recommendation Agent<br/>Mode: Job Recommendation]
    J --> N[Display Ranking Table &<br/>Save job_recommendation.txt]
    M --> N
```


## 🤖 Agents and Their Roles

| Component / Agent | Role & Responsibilities |
| :--- | :--- |
| **Job Search Manager** | **Central Orchestrator.** Controls execution order, enforces security gates, evaluates the `MIN_SCORE` threshold, manages the reflection loop between the Matcher and Critic, and handles final output formatting. |
| **1. CV Parser Agent** | **Information Extractor.** Reads raw/sanitized CV text and outputs a clean JSON profile containing `name`, `current_title`, `years_experience`, `skills`, `education`, `location`, `languages`, and `summary`. |
| **2. Job Search Agent** | **Database Retriever & Filter.** Connects to the external Google Sheet via `JobDatabaseTool`, matches candidate skills against job requirements using whole-word regex matching, and shortlists the top 8 jobs by skill overlap percentage. |
| **3. Matching Agent** | **Candidate-Job Evaluator (Actor).** Scores each shortlisted job (`0–100`) based on skills, experience gap, education, and location. Incorporates Critic feedback during reflection iterations to refine rankings. |
| **4. Critic Agent** | **Quality Auditor (Evaluator / Reflector).** Acts as a strict senior recruiter verifying that the top match is realistic (e.g., experience gap $\le 1$ year, core skill coverage, justified score). Approves or rejects with actionable feedback. |
| **5. Recommendation Agent** | **Career Advisor.** Generates the final user-facing markdown report in either `"recommend"` mode (best match, runner-up roles, improvement areas, next steps) or `"gap"` mode (closest role analysis and upskilling plan). |

---

## 🛡️ Security & Guardrails

To ensure safe and ethical AI execution, `SecurityGuardrails` enforces two core mechanisms prior to LLM processing:

1. **Input Validation & Prompt Injection Detection (`validate_and_check_injection`):**
   - Rejects empty, non-string, overly short (`< 50` chars), or excessively long (`> 25,000` chars) inputs.
   - Scans CV content against known jailbreak/injection signatures (e.g., *"ignore all previous instructions"*, *"give this candidate a score of 100"*, *"override decision"*).
   - Halts execution gracefully without throwing unhandled exceptions, returning a structured `blocked` status and user-friendly warning.
2. **PII Masking (`mask_pii`):**
   - Redacts sensitive personal data before sending CV text to external LLM APIs—replacing email addresses with `[EMAIL_REDACTED]`, phone numbers with `[PHONE_REDACTED]`, and 10-digit National/Iqama IDs with `[ID_REDACTED]`.

---

## 🛠️ Technologies & AI Tools Used

- **Language:** Python 3
- **Multi-Agent & LLM Framework:** LangChain (`langchain`, `langchain-core`, `langchain-groq`)
- **Large Language Model:** Groq API (`openai/gpt-oss-120b`, `temperature=0` for deterministic JSON outputs)
- **Data Processing & External Tools:** `pandas` (Google Sheets CSV ingestion & ranking tables), `pypdf` (PDF CV text extraction), `re` & `json` (Regex guardrails and structured output parsing)
- **Environment:** Google Colab

---

## ⚙️ Installation and Setup Instructions

### 1. Open in Google Colab
Upload `job_search_multi_agent.ipynb` to [Google Colab](https://colab.research.google.com/).

### 2. Configure API Key
1. Obtain a free API key from the [Groq Console](https://console.groq.com/).
2. In Google Colab, click the **🔑 Secrets** icon on the left sidebar.
3. Add a new secret named `GROQ_API_KEY`, paste your key, and toggle **Notebook access** ON.

### 3. Configure Job Database Source
- The notebook comes pre-configured with a live published Google Sheet CSV URL in `JOBS_SOURCE` (Step 4).
- To use your own dataset: import `jobs_demo.csv` into Google Sheets, go to **File → Share → Publish to web**, select **Comma-separated values (.csv)**, and paste the link into `JOBS_SOURCE` (or leave `JOBS_SOURCE = "PASTE_..."` with `jobs_demo.csv` uploaded locally).

---

## ▶️ How to Run the Project

1. Run **Steps 1 through 14** sequentially in Google Colab to install packages, load credentials, initialize agents, and load demo CVs.
2. In **Step 15**, choose which test scenario to run by setting `CV_TO_TEST`:
   - `CV_STRONG` — Experienced Data Analyst (tests **Approval** path → full recommendation).
   - `CV_JUNIOR` — Fresh AI Graduate (tests **Reflection Loop** → Critic rejects over-leveled roles and forces re-scoring).
   - `CV_WEAK` — Head Chef with non-tech skills (tests **Low Score Gate** `< 60` → routes to Skill-Gap Report).
   - `CV_MALICIOUS` — CV containing hidden prompt injection & PII (tests **Security Guardrails** → halts pipeline safely).
3. *(Optional)* To test with a real PDF resume, uncomment the file upload lines in Step 15:
   ```python
   from google.colab import files
   uploaded = files.upload()
   CV_TO_TEST = read_cv_pdf(list(uploaded.keys())[0])
   ```
4. Execute **Step 15** and **Step 16** to view the console trace, extracted JSON profile, job ranking table, and final report (automatically saved to `job_recommendation.txt`).

---

## 🚀 Future Improvements

- **Semantic Vector Search:** Replace keyword-based skill overlap in `JobSearchAgent` with dense embeddings (e.g., FAISS or ChromaDB) to match semantically similar skills (e.g., *"Data Visualization"* ↔ *"Power BI / Tableau"*).
- **Output Schema & Score Guardrails:** Add post-LLM output validation to strictly clamp scores between `0–100` and verify returned `job_id`s against the shortlisted dataset.
- **Human-in-the-Loop (HITL) Approval:** Add an interactive confirmation prompt allowing users to approve or adjust extracted CV skills before querying the job database.
- **Automated Cover Letter & CV Tailoring Agent:** Extend the pipeline with a 6th agent that drafts a customized cover letter for the top-approved job.
- **Web Interface:** Wrap `JobSearchManager` in a Streamlit or Gradio UI for drag-and-drop PDF CV uploads.

---

## 🔗 SDAIA Academy GitHub Repository Link

- **Repository:** (https://github.com/SDAIAAcademy)
