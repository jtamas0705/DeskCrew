# 🖥️ Desk Crew
### Local Personal Productivity Assistant — running fully on NVIDIA Jetson Orin Nano Super

> No cloud. No data leakage. A small team of AI agents that helps you manage tasks, summarise documents, and draft responses — pausing for your approval at every key step.

---

## Stack

| Layer | Tool | Purpose |
|---|---|---|
| Agent Orchestration | [CrewAI](https://github.com/crewAIInc/crewAI) | Agent roles, task delegation, crew lifecycle |
| Retrieval / RAG | [LlamaIndex](https://github.com/run-llama/llama_index) | Indexes your local PDFs, notes, docs |
| Observability | [NVIDIA AgentIQ](https://github.com/NVIDIA/agentiq) | Latency profiling, token usage, tracing |
| Local Inference | [Ollama](https://ollama.com) | Model serving on-device |
| UI | [Gradio](https://gradio.app) | Human-in-the-loop approval interface |
| Hardware | NVIDIA Jetson Orin Nano Super (8 GB) | Edge inference, fully air-gapped |

---

## The Crew

```
Planner      →  Breaks your request into subtasks       (Phi-3 Mini)
Researcher   →  Searches your local docs via RAG        (Phi-3 Mini)
Writer       →  Drafts emails, summaries, task lists    (Llama 3.2 3B)
Reviewer     →  Checks quality · waits for YOUR OK      (Phi-3 Mini)
```

---

## Workflows

### 1 · Morning Briefing
One button. Researcher scans yesterday's notes → Planner prioritises → Writer drafts a briefing → Reviewer presents it for your approval.

### 2 · Document Q&A
Drop a PDF into the watched folder. LlamaIndex indexes it automatically. Ask questions in plain English. Reviewer checks for hallucinations before showing you the answer.

### 3 · Draft & Send
Give a rough instruction. Writer drafts it. Reviewer grades tone and clarity. **Nothing goes out without your explicit sign-off.**

---

## Prerequisites

- NVIDIA Jetson Orin Nano Super (8 GB)
- JetPack 6.x
- Docker (with NVIDIA Container Runtime)
- Python 3.10+
- NVMe SSD recommended

---

## Installation

```bash
# 1. Clone the repo
git clone https://github.com/yourname/desk-crew.git
cd desk-crew

# 2. Install dependencies
pip install crewai llama-index gradio agentiq langchain-community

# 3. Install and start Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 4. Pull models
ollama pull phi3:mini
ollama pull llama3.2:3b

# 5. Set Ollama to keep both models warm
export OLLAMA_MAX_LOADED_MODELS=2
```

---

## Quick Start

```python
from crewai import Agent, Task, Crew
from langchain_community.llms import Ollama

# Point CrewAI at your local Ollama instance
llm_small = Ollama(model="phi3:mini",    base_url="http://localhost:11434")
llm_write = Ollama(model="llama3.2:3b", base_url="http://localhost:11434")

planner = Agent(
    role="Planner",
    goal="Break the user request into clear subtasks",
    backstory="You are a meticulous project planner.",
    llm=llm_small
)

writer = Agent(
    role="Writer",
    goal="Produce clear, concise written output",
    backstory="You are a skilled professional writer.",
    llm=llm_write
)

crew = Crew(
    agents=[planner, writer],
    tasks=[...],
    process="sequential"
)

result = crew.kickoff()
```

---

## Project Structure

```
desk-crew/
├── agents/
│   ├── planner.py
│   ├── researcher.py
│   ├── writer.py
│   └── reviewer.py
├── workflows/
│   ├── morning_briefing.py
│   ├── document_qa.py
│   └── draft_and_send.py
├── rag/
│   └── indexer.py          # LlamaIndex setup, watched folder logic
├── ui/
│   └── app.py              # Gradio human-in-the-loop interface
├── observability/
│   └── agentiq_wrapper.py  # AgentIQ profiling setup
├── config/
│   └── models.yaml         # Model assignments per agent
├── docs/                   # Drop your PDFs/notes here
├── main.py
├── requirements.txt
└── README.md
```

---

## Build Milestones

| Week | Goal | What you learn |
|---|---|---|
| 1 | Ollama + CrewAI talking | Multi-agent handoff, basic plumbing |
| 2 | Add LlamaIndex | RAG over your own documents |
| 3 | Human-in-the-loop UI | Approval gates with Gradio |
| 4 | Wrap AgentIQ | Profiling, latency tuning, token optimisation |

---

## Hardware Tips

- **Keep models warm** — `OLLAMA_MAX_LOADED_MODELS=2` prevents cold-load latency between agent calls
- **Use NVMe for vector store** — store the LlamaIndex index on SSD, not eMMC
- **Start small** — `num_ctx=2048` per agent until you've profiled your baseline
- **Crew size** — 2–3 agents for weeks 1–2, add the 4th once the pipeline is stable
- **Code execution** — if any agent runs code, use `code_execution_mode: safe` (Docker sandbox)

---

## Model Recommendations

| Need | Model | Why |
|---|---|---|
| Fast reasoning (Planner/Reviewer) | `phi3:mini` | Low memory, good instruction following |
| Writing quality | `llama3.2:3b` | Better prose, fits in 8 GB alongside Phi-3 |
| Better reasoning (if you can wait) | `mistral:7b-q4` | Noticeably smarter, slower inference |

---

## Observability with AgentIQ

Wrap your crew with AgentIQ to get real-time telemetry on which agent is slow, which burns the most tokens, and where to optimise first:

```python
from agentiq import profile

with profile() as session:
    result = crew.kickoff()

session.report()  # prints per-agent latency + token usage
```

---

## Licence

MIT — do whatever you like with it.

---

*Desk Crew · Built on CrewAI · LlamaIndex · NVIDIA AgentIQ · Ollama · Jetson Orin Nano Super*
