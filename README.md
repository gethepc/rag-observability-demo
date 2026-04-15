# RAG Observability Demo

> **Demonstration of a RAG-based knowledge retrieval system applied to IT observability and incident intelligence.**

[![Status](https://img.shields.io/badge/status-demo--architecture-blue)](https://github.com/gethepc/rag-observability-demo)
[![Stack](https://img.shields.io/badge/stack-LangChain%20%7C%20Ollama%20%7C%20pgvector%20%7C%20FastAPI-brightgreen)](https://github.com/gethepc/rag-observability-demo)

---

## Overview

This repo documents the **RAG (Retrieval-Augmented Generation) architecture** used inside the Intelligent Healing System (IHS) to enable AI-powered incident resolution using historical knowledge.

The design demonstrates how to apply RAG to operational data — incident records, runbooks, post-mortems — so an LLM can answer operational questions grounded in your organisation's actual knowledge base, not generic training data.

---

## The Core Problem RAG Solves in Ops

A standard LLM knows nothing about:
- Your specific system topology
- Your past incidents and how they were resolved
- Your internal runbooks and SOPs
- Your application-specific error patterns

RAG bridges this gap by **retrieving relevant context from your own knowledge corpus** before generating a response.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  RAG Pipeline                       │
│                                                     │
│  [Query: "CPU spike on app-server-03"]              │
│       │                                             │
│       ▼                                             │
│  ┌──────────┐    ┌──────────────────────────────┐   │
│  │ Embedding│───▶│  Vector Search               │   │
│  │ Model    │    │  (PostgreSQL + pgvector)      │   │
│  └──────────┘    └──────────┬───────────────────┘   │
│                             │ Top-k similar docs    │
│                             ▼                       │
│                  ┌──────────────────────────────┐   │
│                  │  Context Assembly             │   │
│                  │  (retrieved chunks +         │   │
│                  │   original query)            │   │
│                  └──────────┬───────────────────┘   │
│                             │                       │
│                             ▼                       │
│                  ┌──────────────────────────────┐   │
│                  │  LLM (Ollama — local)         │   │
│                  │  Generate grounded response   │   │
│                  └──────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

---

## Knowledge Corpus

The system ingests and vectorises:

| Source | Content |
|---|---|
| Incident records | Past incidents, symptoms, resolution steps |
| Post-mortems | Root cause analysis, contributing factors |
| Runbooks | Step-by-step operational procedures |
| Alert definitions | What each alert means, typical causes |
| System topology docs | Service dependencies, architecture diagrams |

---

## Key Design Decisions

### Why local LLM (Ollama)?
- Data sovereignty — no incident data leaves the environment
- No per-token API costs at scale
- Models: Llama 3, Mistral, or Mixtral depending on hardware

### Why pgvector over a dedicated vector DB?
- PostgreSQL already present in most enterprise stacks
- Transactional consistency with metadata filtering
- Simpler ops — one fewer system to manage

### Chunking Strategy
- Incident records: chunked by resolution step
- Runbooks: chunked by section heading
- Chunk size: 512 tokens with 50-token overlap
- Metadata preserved: source, date, system, severity

### Retrieval
- Top-k = 5 chunks retrieved per query
- Similarity: cosine distance on embeddings
- Metadata filters applied pre-retrieval (e.g., filter by system name)

---

## API Layer

```
POST /query
{
  "question": "What caused the database connection pool exhaustion last quarter?",
  "filters": {"system": "payments-db", "severity": "P1"}
}

Response:
{
  "answer": "Based on 3 matched incidents...",
  "sources": ["INC-4421", "postmortem-2024-09.md"],
  "confidence": 0.87
}
```

---

## Technology Stack

| Component | Technology |
|---|---|
| Embedding model | `nomic-embed-text` via Ollama |
| LLM | Llama 3 / Mistral via Ollama |
| Vector store | PostgreSQL + pgvector |
| Document store | MongoDB |
| Orchestration | LangChain |
| API | FastAPI |
| Containerisation | Docker |

---

## Repository Structure

```
rag-observability-demo/
├── docs/
│   ├── rag-architecture.md
│   ├── chunking-strategy.md
│   ├── embedding-model-selection.md
│   └── api-design.md
├── examples/
│   ├── sample-incident-corpus.json
│   ├── sample-runbook.md
│   └── query-examples.md
├── assets/
│   └── rag-pipeline-diagram.png
└── README.md
```

---

## Related Projects

- [ihs-ai-ops-platform](https://github.com/gethepc/ihs-ai-ops-platform) — full platform where this RAG system is deployed

---

## Author

**Prashant Chauhan** — Conceptualised · Designed · Built · Integrated into IHS  
GitHub: [@gethepc](https://github.com/gethepc)  
LinkedIn: [prashant-chauhan-2b5a0b11](https://www.linkedin.com/in/prashant-chauhan-2b5a0b11/)
