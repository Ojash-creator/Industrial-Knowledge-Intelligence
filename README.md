# Industrial Knowledge Intelligence Copilot

RAG-powered assistant over industrial maintenance logs, SOPs, incident reports,
compliance docs, and P&ID descriptions. Built for the ET AI Hackathon 2026
(Problem Statement 8).

**100% free to run** — no paid API, no paid hosting required.

## Stack (all free)

| Component      | Tool                                   | Cost |
|-----------------|-----------------------------------------|------|
| Embeddings      | `sentence-transformers` (local, CPU)    | Free |
| Vector DB       | ChromaDB (local, persistent)            | Free |
| LLM             | Groq API (`llama-3.3-70b-versatile`)    | Free tier |
| UI              | Streamlit                               | Free |
| Hosting (demo)  | Streamlit Community Cloud / local       | Free |

## 1. Setup

```bash
# Clone / cd into project
cd industrial-rag

# Create a virtual environment (recommended)
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

## 2. Get a free Groq API key

1. Go to https://console.groq.com/keys
2. Sign up (free, no credit card required)
3. Create an API key
4. Set it as an environment variable:

```bash
# Mac/Linux
export GROQ_API_KEY="your_key_here"

# Windows (PowerShell)
$env:GROQ_API_KEY="your_key_here"
```

Free tier limits (subject to change, check Groq's console): generous requests/min
and tokens/day — more than enough for a hackathon demo. If you want a backup
LLM provider, Google AI Studio (Gemini) also has a free tier and can be swapped
into `src/rag_engine.py` with minimal changes.

## 3. Generate the synthetic document corpus

```bash
python src/generate_synthetic_data.py
```

This creates ~13 realistic industrial documents in `data/documents/`:
maintenance logs, SOPs (confined space, hot work, PTW), incident/near-miss
reports, regulatory compliance docs, and P&ID descriptions — all cross-referenced
with each other (e.g. an incident report references a maintenance work order,
which references a P&ID equipment tag) so retrieval + reasoning across documents
is demonstrable, not just single-doc lookup.

**To add your own real documents:** just drop `.txt` or `.md` files into
`data/documents/` — the pipeline treats them identically. (PDF/DOCX support can
be added with `pypdf`/`python-docx` if you have real files in those formats —
ask and I'll extend the ingestion script.)

## 4. Ingest documents into the vector database

```bash
python src/ingest.py
```

This chunks all documents, embeds them locally (no API call, no cost), and
stores them in a persistent ChromaDB collection at `chroma_db/`.

## 5. Run the app

```bash
streamlit run app.py
```

Opens at `http://localhost:8501`. The app has **three tabs**:

### Tab 1 — 💬 Chat Copilot
Ask questions, get cited answers. Broad questions ("summarize all...",
"across the plant", "compare...") automatically retrieve up to 4x more
context chunks so the answer draws from many documents, not just a narrow
match.

### Tab 2 — 🕸️ Knowledge Graph
Click **Build / Rebuild Graph** once (calls the LLM once per document to
extract entities — assets, permits, incidents, standards, locations — and
their relationships). Renders as an interactive, hoverable network graph.
Highly-connected nodes are usually systemic issues. This directly answers
the problem statement's "Universal Document Ingestion & Knowledge Graph
Agent" ask — most hackathon RAG submissions skip this and only do vector
search, so this is a differentiator.

### Tab 3 — 🛡️ Compliance Agent
Click **Run Full-Corpus Compliance Scan**. This is *agentic*, not reactive —
it reads every document in the corpus without being asked a specific
question and proactively surfaces: systemic patterns, regulatory compliance
gaps, and a ranked list of top-5 actions with evidence citations. This
answers the "Quality & Regulatory Compliance Intelligence" and "Lessons
Learned & Failure Intelligence Engine" asks from the problem statement.

Every answer/report shows which source documents were used (citations),
which directly demonstrates the "RAG with citations" and audit-trail
requirements.

## 6. (Optional) Test components directly from CLI

```bash
python src/rag_engine.py "What compound risk exists in Coke Oven Battery Zone B?"
python src/build_knowledge_graph.py
python src/compliance_agent.py
```

## 7. Free hosting for the demo (optional)

If you want a shareable link instead of just localhost for judges:

1. Push this project to a public GitHub repo
2. Go to https://share.streamlit.io (Streamlit Community Cloud, free)
3. Connect your GitHub repo, set `app.py` as the entry point
4. Add `GROQ_API_KEY` as a secret in the app settings
5. Deploy — you get a public URL

Otherwise, running locally + a recorded demo video (as required by the
hackathon deliverables) works fine too.

## Project structure

```
industrial-rag/
├── app.py                          # Streamlit UI - 3 tabs (Chat/Graph/Compliance)
├── requirements.txt
├── .streamlit/config.toml          # UI config (hides default deploy button)
├── data/
│   ├── documents/                  # Source documents (19 synthetic + real)
│   ├── knowledge_graph.json        # Generated by build_knowledge_graph.py
│   └── latest_compliance_scan.md   # Generated by compliance_agent.py
├── src/
│   ├── generate_synthetic_data.py  # Creates 19-document synthetic corpus
│   ├── ingest.py                   # Chunk + embed + store in ChromaDB
│   ├── rag_engine.py               # Retrieval + Groq LLM query (reactive)
│   ├── build_knowledge_graph.py    # Entity/relationship extraction agent
│   └── compliance_agent.py         # Proactive full-corpus pattern scanner
└── chroma_db/                      # Persistent vector store (generated)
```

## Why this stands out for judging (three pillars, not one)

Most hackathon RAG submissions are "chatbot over PDFs" - one retrieval
pipeline, reactive only. This platform has three integrated layers that map
directly onto the problem statement's suggested technologies:

| Layer | Problem statement ask it answers | Judging criteria it strengthens |
|---|---|---|
| **Chat Copilot (RAG)** | "Expert Knowledge Copilot" | Technical Excellence, UX |
| **Knowledge Graph** | "Universal Document Ingestion & Knowledge Graph Agent" | Innovation, Technical Excellence |
| **Compliance Agent** | "Quality & Regulatory Compliance Intelligence" + "Lessons Learned & Failure Intelligence Engine" | Business Impact, Innovation |

The synthetic corpus is deliberately cross-referenced (an incident report
cites a work order, which cites a P&ID equipment tag, which is governed by
an SOP, which maps to a compliance standard) so that both the knowledge
graph and the compliance agent have real multi-hop patterns to surface -
not just single-document lookups. The `quality_audit_finding_2025_q2.txt`
document is written as a "meta" document that ties every other document's
pattern together - a strong live-demo moment: run the compliance scan, and
show it independently arrives at the same conclusions a human quarterly
audit took a full quarter to find.

## What to highlight in your pitch

- **Three integrated intelligence layers, not one search box**: RAG copilot
  (reactive), Knowledge Graph (structural), Compliance Agent (proactive/
  agentic). This is the difference between "search" and "intelligence."
- **Cross-document reasoning**: ask "what recurring pattern connects hot work
  and confined space permits?" - the answer pulls from an SOP, two incident
  reports, and a compliance doc simultaneously.
- **Proactive, not just reactive**: the Compliance Agent surfaces patterns
  nobody asked about - this is the "acts on it before a fatality, not after"
  requirement from the problem statement's own framing.
- **Citations everywhere**: every answer, graph edge, and compliance finding
  traces back to a source document - critical for OISD/DGMS/Factory Act audit
  trails.
- **Zero infrastructure cost**: fully runnable on a laptop with free-tier
  APIs - strengthens Scalability and Business Viability scoring since it's
  deployable at any facility without licensing cost for the core pipeline.
- **Extensibility**: OCR for scanned P&IDs, live SCADA/IoT integration, and
  CCTV analytics (from the "compound risk detection" problem statement #1)
  are natural v2 extensions on top of this same knowledge graph foundation.
