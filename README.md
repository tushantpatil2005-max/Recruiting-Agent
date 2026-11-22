# AI Recruiting Agent

**Recruitbots AI — Agentic AI Engineer · Challenge 2 (Intermediate)**

One implementation file: [`recruiting_agent.py`](recruiting_agent.py).

Reviews resumes, extracts candidate fields, matches them to job descriptions, scores fit with explicit reasoning, and recommends next actions. Uses **tool calling** against an internal candidate store and job catalog. **No fine-tuned models.**

Customer-facing summaries and recommendations carry no machine-origin labels.

---

## Quick start

```bash
python recruiting_agent.py --check

# Evaluate an internal candidate for a role
python recruiting_agent.py --candidate cand_001 --job backend_engineer

# Rank internal pool for a job
python recruiting_agent.py --rank --job data_analyst

# Parse a local resume file
python recruiting_agent.py --resume-file path/to/resume.txt --job frontend_engineer

# HTTP API
pip install fastapi uvicorn
python recruiting_agent.py --serve
# POST /evaluate  POST /rank  GET /jobs  GET /tools  GET /health
```

---

## Architecture

```text
Input: resume text | candidate_id | search query  +  job_id | job title
                    │
                    ▼
            RecruitingAgent
                    │
    ┌───────────────┼───────────────┐
    ▼               ▼               ▼
parse_resume  search_candidates  get_job_description
                    │
                    ▼
               score_match
                    │
                    ▼
     Structured JSON evaluation
     (scores, reasoning, next actions, summary)
```

Each evaluation records tool steps (thought, action, arguments, observation) so the path is inspectable.

---

## Success criteria

| Criterion | Implementation |
| --- | --- |
| Parses resume content accurately | `parse_resume` extracts name, email, skills, years, education, titles |
| Generates candidate scores | Skills / experience / education / overall (0–100) |
| Reasoning for recommendations | Human-readable `reasoning` plus matched / missing skill lists |
| Multiple job roles | `backend_engineer`, `frontend_engineer`, `data_analyst`, `devops_engineer` |
| Structured JSON output | Stable schema from `/evaluate` and `/rank` |

---

## Constraints

| Constraint | How it is met |
| --- | --- |
| ≥2 tools | Four tools: `parse_resume`, `search_candidates`, `get_job_description`, `score_match` |
| Evaluation scoring | Weighted fit: 55% skills, 30% experience, 15% education |
| No fine-tuned models | Deterministic parsers and scorers only; no trained ranker |

---

## Tools

| Tool | Purpose |
| --- | --- |
| `parse_resume` | Turn free-text resume into structured fields |
| `search_candidates` | Query the internal candidate database |
| `get_job_description` | Load role requirements by id or title |
| `score_match` | Score a profile against a job; emit recommendation |

---

## Scoring and recommendations

| Overall | Recommendation | Typical next actions |
| --- | --- | --- |
| ≥80 and no missing required skills | `advance` | Technical screen; share scorecard |
| ≥65 with ≤2 missing required | `consider` | Clarify gaps; optional take-home |
| Otherwise | `pass` | Polite decline; tag for future roles |

---

## Example JSON (shape)

```json
{
  "id": "eval_…",
  "status": "complete",
  "candidate": {
    "name": "Maya Chen",
    "email": "maya.chen@example.com",
    "skills": ["python", "fastapi", "postgresql"],
    "years_experience": 5,
    "education": "B.S. Computer Science, University of Washington"
  },
  "job": { "id": "backend_engineer", "title": "Backend Engineer" },
  "scores": { "skills": 91.0, "experience": 82.0, "education": 90.0, "overall": 88.1 },
  "recommendation": "advance",
  "reasoning": "Matched 5/5 required skills …",
  "next_actions": ["Schedule a technical screen", "…"],
  "summary": "Maya Chen is evaluated for Backend Engineer …",
  "tools_used": ["get_job_description", "parse_resume", "score_match", "search_candidates"]
}
```

---

## Unmarked output

`strip_machine_labels` removes phrases such as “AI-generated”, “powered by AI”, or “as an AI…” from summaries and reasoning. Internal `steps` remain available for auditors.

---

## Self-check

`python recruiting_agent.py --check` asserts:

- Resume parse recovers name, skills, and years  
- Backend evaluation for a strong Python candidate scores high and recommends advance/consider  
- Same frontend candidate scores higher on frontend than backend (multi-role)  
- Raw resume path works for DevOps  
- Ranking returns ordered overall scores  
- ≥2 tools used; no origin labels in JSON  

---

## File layout

```text
recruiting_agent.py          # agent, tools, scoring, API, self-check
README_recruiting_agent.md   # this document
```
