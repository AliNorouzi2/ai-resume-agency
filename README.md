<div align="center">

# AI Recruitment Multi-Agent System

### Automated resume screening & job-matching powered by local LLMs

![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-App-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-Llama%203.2-000000?style=for-the-badge&logo=ollama&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-Database-07405E?style=for-the-badge&logo=sqlite&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

</div>

---

## Overview

Given a candidate's resume (PDF or raw text), this system automatically:

- 📄 **Extracts** structured information from the resume
- 🔍 **Analyzes** skills, experience level, and background
- 🎯 **Matches** the candidate against open job positions in a database
- 👥 **Screens** the candidate for qualification and cultural fit
- 💡 **Recommends** a final hiring decision with clear next steps

All powered by a coordinated team of AI agents, running entirely on a **local LLM** — no external API keys or cloud costs required.

---

## Architecture

```
                        ┌────────────────────┐
                        │  OrchestratorAgent  │
                        └──────────┬──────────┘
                                   │
        ┌───────────┬─────────────┼─────────────┬────────────┐
        ▼           ▼             ▼             ▼            ▼
  📄 Extractor → 🔍 Analyzer → 🎯 Matcher → 👥 Screener → 💡 Recommender
```

All agents inherit from a shared `BaseAgent` class, which handles communication with the local LLM (Ollama) and safe JSON parsing of model responses.

### Agents

| Agent | Emoji | Responsibility |
|---|:---:|---|
| **ExtractorAgent** | 📄 | Extracts raw text from a resume PDF (`pdfminer.six`) and structures it via the LLM. |
| **AnalyzerAgent** | 🔍 | Determines technical skills, years of experience, education level, seniority, achievements, and domain expertise. |
| **MatcherAgent** | 🎯 | Queries a SQLite job database and scores candidates against open positions. |
| **ScreenerAgent** | 👥 | Evaluates qualification alignment, experience relevance, skill match, and red flags. |
| **RecommenderAgent** | 💡 | Synthesizes all prior stages into a final recommendation with confidence level and next steps. |
| **OrchestratorAgent** | 🧠 | Coordinates the end-to-end workflow and aggregates results across all agents. |

---

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| 🐍 **Python 3.12** | Core implementation language |
| 🎈 **Streamlit** | Interactive web UI (`app.py`) |
| 🦙 **Ollama (Llama 3.2)** | Local LLM inference for all reasoning/analysis tasks |
| 🔌 **OpenAI Python SDK** | OpenAI-compatible client pointed at the local Ollama server |
| 📑 **pdfminer.six** | PDF text extraction from resumes |
| 🗄️ **SQLite3** | Local job listings database used by the matching stage |
| ⚡ **asyncio** | Asynchronous agent execution model |

---

## 📁 Project Structure

```
.
├── app.py                    # Streamlit web application entry point
├── requirements.txt          # Project dependencies
├── __init__.py
├── base_agent.py              # Shared base class: LLM querying + safe JSON parsing
├── extractor_agent.py         # Resume text extraction & structuring
├── analyzer_agent.py          # Candidate profile analysis
├── matcher_agent.py           # Job matching against SQLite job database
├── screener_agent.py          # Candidate screening
├── recommender_agent.py       # Final recommendation generation
├── orchestrator.py            # Workflow coordination across all agents
└── profile_enhancer_agent.py  # (Legacy/alternate) profile summary enhancement
```

---

## ⚙️ Prerequisites

- Python 3.12+
- [Ollama](https://ollama.com) installed and running locally with the `llama3.2` model pulled:
  ```bash
  ollama pull llama3.2
  ollama serve
  ```
- A SQLite `jobs` database accessible via the `db.database.JobDatabase` module (used by `MatcherAgent`)

---

## Installation

```bash
pip install -r requirements.txt
```

---

## ▶️ Usage

Run the Streamlit app:

```bash
streamlit run app.py
```

This launches an interactive web interface where you can upload a resume and view the full extraction → analysis → matching → screening → recommendation pipeline in action.

### Programmatic usage

```python
import asyncio
from orchestrator import OrchestratorAgent

async def main():
    orchestrator = OrchestratorAgent()
    result = await orchestrator.process_application({
        "file_path": "path/to/resume.pdf"
        # or: "text": "raw resume text..."
    })
    print(result["final_recommendation"])

asyncio.run(main())
```

The orchestrator runs the full pipeline and returns a `workflow_context` dictionary containing the output of every stage.

---

## Notes

- The `MatcherAgent` requires a `db/database.py` module exposing a `JobDatabase` class with a `db_path` attribute pointing to a SQLite database containing a `jobs` table (`title`, `company`, `location`, `type`, `experience_level`, `salary_range`, `description`, `requirements`, `benefits`).
- Inter-agent messages currently pass data as Python-literal strings (`str(dict)` / `eval(...)`), which is convenient for local prototyping but should be replaced with proper JSON serialization before any production or externally-facing deployment.
- `profile_enhancer_agent.py` depends on the `swarm` framework and represents an earlier/alternate design not wired into the current `OrchestratorAgent` pipeline.

---
