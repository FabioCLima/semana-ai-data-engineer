<div align="center">

# ShopAgent — AI Data Engineering in Production

**Multi-agent AI system for e-commerce: autonomous SQL + semantic routing, built live in 4 nights**

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat&logo=python&logoColor=white)](https://python.org)
[![LangChain](https://img.shields.io/badge/LangChain-0.3+-1C3C3C?style=flat&logo=langchain&logoColor=white)](https://langchain.com)
[![CrewAI](https://img.shields.io/badge/CrewAI-0.100+-FF6B6B?style=flat)](https://crewai.com)
[![Qdrant](https://img.shields.io/badge/Qdrant-vector--db-DC143C?style=flat)](https://qdrant.tech)
[![Anthropic](https://img.shields.io/badge/Claude-Anthropic-orange?style=flat)](https://anthropic.com)
[![Docker](https://img.shields.io/badge/Docker-compose-2496ED?style=flat&logo=docker&logoColor=white)](https://docker.com)
[![LangFuse](https://img.shields.io/badge/LangFuse-observability-blueviolet?style=flat)](https://langfuse.com)

</div>

---

## What This Project Demonstrates

A production-grade AI agent system that answers e-commerce business questions by autonomously deciding **which data store to query** — structured SQL for exact metrics, or vector search for semantic understanding of customer reviews.

This is not a tutorial follow-along. It is a complete system built from scratch, covering the full AI Data Engineering stack:

- **Data generation** — synthetic e-commerce data via ShadowTraffic (500 customers, 200 products, 5,000 orders, 200+ Portuguese reviews)
- **Data validation** — Pydantic v2 models with structured outputs from Claude
- **Context Engineering** — LlamaIndex RAG pipeline with FastEmbed embeddings on Qdrant
- **Autonomous agent** — LangGraph ReAct agent routing between SQL and vector search
- **Multi-agent crew** — CrewAI with 3 specialist agents working sequentially
- **Observability** — LangFuse traces showing latency, token cost, and quality per agent
- **Quality assurance** — DeepEval testing tool correctness and answer relevancy
- **Cloud portability** — same codebase runs locally (Docker) or in cloud (Supabase + Qdrant Cloud)

---

## Architecture

```
Question: "What's the average order value from customers complaining about delivery?"

┌─────────────────────────────────────────────────────────────────┐
│                        SHOPAGENT CREW                           │
│                                                                 │
│  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐   │
│  │ AnalystAgent  │    │ ResearchAgent │    │ ReporterAgent │   │
│  │  SQL queries  │ →  │ Vector search │ →  │  Synthesizes  │   │
│  │  exact KPIs   │    │  sentiment    │    │  final answer │   │
│  └──────┬────────┘    └──────┬────────┘    └───────────────┘   │
│         │                   │                                   │
└─────────┼───────────────────┼───────────────────────────────────┘
          │                   │
          ▼                   ▼
  ┌───────────────┐   ┌───────────────┐
  │  THE LEDGER   │   │  THE MEMORY   │
  │  PostgreSQL   │   │    Qdrant     │
  │  (Supabase)   │   │ (Vector DB)   │
  │               │   │               │
  │ orders        │   │ reviews       │
  │ customers     │   │ embeddings    │
  │ products      │   │ sentiment     │
  └───────────────┘   └───────────────┘
```

**The Ledger** answers: *"What was total revenue in São Paulo last month?"* → SQL → R$ 127.430,50

**The Memory** answers: *"Who is complaining about slow delivery?"* → Vector search → 23 similar reviews

**Both together** answers: *"Average order value of customers complaining about delivery?"* → Cross-store intelligence

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Data Generation | [ShadowTraffic](https://shadowtraffic.io) | Declarative synthetic e-commerce data |
| Validation | Pydantic v2 + Anthropic tool use | Typed models + structured LLM outputs |
| RAG Pipeline | LlamaIndex + FastEmbed | Document ingestion → embeddings → retrieval |
| Vector Store | Qdrant | Semantic search on customer reviews |
| Relational Store | PostgreSQL / Supabase | Transactional e-commerce data |
| Agent Framework | LangGraph (ReAct) | Autonomous tool routing |
| Multi-Agent | CrewAI | 3-specialist crew with sequential process |
| Tool Protocol | MCP (Model Context Protocol) | Standardized tool exposure for agents |
| Chat Interface | Chainlit | Conversational UI with streaming |
| Observability | LangFuse | Traces, cost, latency per agent |
| Quality | DeepEval | Tool correctness + answer relevancy tests |
| LLM | Claude (Anthropic) | Reasoning, structured outputs, tool use |

---

## Day-by-Day Build

| Day | Theme | What Was Built |
|-----|-------|----------------|
| **Day 1** — INGEST | Data foundation | ShadowTraffic generators, Pydantic models, structured outputs via Claude |
| **Day 2** — CONTEXTUALIZE | RAG + Context Eng | LlamaIndex pipeline, Qdrant indexing, MCP for Postgres + Qdrant |
| **Day 3** — AGENT | Autonomous routing | LangGraph ReAct agent deciding SQL vs vector, Chainlit chat UI |
| **Day 4** — MULTI-AGENT | Production system | CrewAI crew, DeepEval tests, LangFuse traces, cloud migration |

---

## Quickstart (Local)

**Prerequisites:** Docker, Python 3.11+, Anthropic API key, [ShadowTraffic free trial](https://shadowtraffic.io)

```bash
# 1. Clone and configure
git clone https://github.com/FabioCLima/semana-ai-data-engineer.git
cd semana-ai-data-engineer
cp .env.example .env
# Set ANTHROPIC_API_KEY in .env

# 2. Start data infrastructure
cd gen
cp .env.example .env && cp license.env.example license.env
docker compose up -d   # Postgres :5432 | Qdrant :6333 | ShadowTraffic

# 3. Python environment
cd ..
python -m venv .venv && source .venv/bin/activate
pip install -r src/requirements.txt

# 4. Run the agent
export PYTHONPATH=src
python -m day2.index_reviews          # Index reviews into Qdrant
chainlit run src/day3/chainlit_app.py # Chat with ShopAgent
```

---

## Data Model

```sql
customers (500 records): customer_id, name, email, city, state, segment
products  (200 records): product_id, name, category, price, brand
orders  (5,000 records): order_id, customer_id FK, product_id FK, qty, total, status, payment
reviews   (200+ JSONL):  review_id, order_id FK, rating, comment (PT-BR), sentiment
```

---

## Project Structure

```
semana-ai-data-engineer/
├── gen/                    # Docker: Postgres + Qdrant + ShadowTraffic
│   ├── docker-compose.yml
│   ├── shadowtraffic.json  # Declarative data generators
│   └── init.sql
├── src/
│   ├── day1/               # Pydantic models + structured outputs
│   ├── day2/               # LlamaIndex RAG + SQL queries + hybrid
│   ├── day3/               # LangGraph agent + Chainlit app
│   └── day4/               # CrewAI crew + DeepEval + LangFuse
├── prompts/                # 44 sequenced prompts (11 per day)
└── docs/                   # Architecture specs + runbooks
```

---

## Skills Demonstrated

`Pydantic v2` · `LangChain` · `LangGraph` · `CrewAI` · `LlamaIndex` · `Qdrant` · `PostgreSQL` · `Docker` · `FastAPI` · `Anthropic API` · `MCP Protocol` · `LangFuse` · `DeepEval` · `Chainlit` · `RAG` · `Context Engineering` · `Multi-Agent Systems` · `Python` · `MLOps`

---

<div align="center">

Built during **Semana AI Data Engineer 2026** · AIDE Brasil  
*"O que eu consigo fazer agora que não conseguia antes?"*

</div>
