<img width="956" height="446" alt="Screenshot 2026-10-23 181351" src="https://github.com/user-attachments/assets/9af14a5b-8f8f-4574-a513-1eed9e57601f" />



# Ontology Engine v5

**An AI-powered knowledge graph builder and research assistant for autonomous mobility domains.**

Built by Tsegai Yhdego, Ph.D. | R-SEAT Center, FAMU-FSU | USDOT-Funded Research

---

## Overview

The Ontology Engine is a 6-agent AI pipeline that ingests research documents and URLs,
extracts entities and relationships using LLMs, builds an interactive knowledge graph,
and enables semantic querying with AI-synthesized answers.

Designed for researchers in intelligent transportation, UAV systems, autonomous vehicles,
and related domains.

---

## Architecture
<img width="606" height="383" alt="image" src="https://github.com/user-attachments/assets/929c94ea-9ba6-4c56-a0ea-b1a124ecf148" />

## Agents

| Agent | File | Role |
|-------|------|------|
| Ingestor | `agents/ingestor.py` | Fetches and chunks text from URLs, files, or raw text |
| Extractor | `agents/groq_agent.py` | Calls Groq LLM to extract nodes and edges as JSON |
| GraphStore | `agents/graph_store.py` | Stores nodes/edges, deduplicates, persists to disk |
| Planner | `agents/planner.py` | Generates multi-step research plans from a topic |
| Query | `agents/query.py` | Keyword-based graph traversal |
| Synthesizer | `agents/groq_agent.py` | LLM-synthesized answers from graph context |

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Backend | FastAPI + Uvicorn |
| LLM | Groq API — `qwen/qwen3.8-27b` |
| Graph storage | In-memory dict + `/data/graph.json` |
| Frontend | Vanilla JS + HTML5 (single file) |
| Tunnel | pyngrok (Google Colab) |
| Runtime | Google Colab (Python 3.10) |

---

## Features

- **5-panel UI**: Text Ingest, URL Ingest, Graph Viewer, Research Planner, Query
- **Research Planner**: AI generates a 5-step research plan with Wikipedia links
- **One-click ingest**: Each planner step has an Ingest button with live progress polling
- **Interactive graph**: D3.js force-directed visualization with node filtering
- **LLM Query**: Ask questions, get synthesized answers grounded in the graph
- **Cross-domain linking**: Automatically links concepts across ingested documents
- **Persistent graph**: Graph survives server restarts via `graph.json`

---

## Setup (Google Colab)

### 1. Install dependencies
```python
!pip install fastapi uvicorn pyngrok groq pdfminer.six pypdf -q
```

### 2. Set API keys
```python
import os
os.environ['GROQ_API_KEY'] = 'your_groq_api_key_here'
os.environ['NGROK_AUTH_TOKEN'] = 'your_ngrok_token_here'
```

### 3. Start the server
```python
import subprocess, threading
proc = subprocess.Popen(
    ['python', '-m', 'uvicorn', 'server:app', '--host', '0.0.0.0', '--port', '8006'],
    cwd='/content'
)
```

### 4. Start ngrok tunnel
```python
from pyngrok import ngrok
ngrok.set_auth_token(os.environ['NGROK_AUTH_TOKEN'])
tunnel = ngrok.connect(8006)
print("App URL:", tunnel.public_url)
```

---

## API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/ingest/text` | Ingest raw text |
| POST | `/ingest/url` | Ingest a URL (async, subprocess worker) |
| POST | `/ingest/file` | Ingest uploaded file (PDF or text) |
| GET | `/graph` | Full graph (nodes + edges) |
| GET | `/health` | Node/edge/doc counts |
| GET | `/sources` | Per-document node counts |
| GET | `/progress/{doc_name}` | Ingest progress for a document |
| POST | `/query` | Keyword graph query |
| POST | `/query/llm` | LLM-synthesized answer |
| POST | `/plan` | Generate research plan |
| POST | `/clear` | Clear graph |

---

## File Structure
<img width="546" height="312" alt="image" src="https://github.com/user-attachments/assets/99698e4c-c8e0-4bae-9996-cec9a00e91f3" />

---

## Known Limitations & Next Steps

- **0-node docs**: Some Wikipedia pages return empty extraction — Groq occasionally
  returns malformed JSON under rate limit pressure. Fix: retry logic + response validation.
- **OTPM limit**: Groq free tier allows 1000 output tokens/min. Ingesting many docs
  sequentially requires 30s gaps. Fix: upgrade tier or switch to `llama-3.1-8b-instant`.
- **Query matching**: Keyword-based — misses semantic similarity. Next: embed nodes
  and use cosine similarity for retrieval.
- **No auth**: Single-user, no authentication. For multi-user: add session management.

---

## Research Context

Developed as a research tool for the **R-SEAT Center** (Research in Sustainable and
Equitable Autonomous Transportation) at FAMU-FSU College of Engineering, supporting
USDOT-funded work on intelligent transportation systems, UAV traffic management,
and autonomous mobility optimization.

**Related domains ingested:**
- Unmanned Aircraft Systems (UAS) and UTM
- Urban Air Mobility (UAM) and eVTOL
- Autonomous vehicles and robotics
- Airspace management and aviation safety

---

## Security Notes

- API keys are stored as environment variables only — never hardcoded
- `GROQ_API_KEY` and `GEMINI_API_KEY` loaded via `os.environ`
- ngrok tunnel is session-scoped and expires when Colab resets

---

## License

Research use. Built at FAMU-FSU R-SEAT Center.
Contact: tsegaiyhdego@gmail.com
"""

with open('/content/README.md', 'w') as f:
    f.write(readme)

import os
print(f"✅ README.md — {os.path.getsize('/content/README.md')/1024:.1f} KB")
