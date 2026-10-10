# Research Copilot

A bilingual (English / Bahasa Indonesia) **verifiable** AI research assistant, built from scratch,
level by level, to learn AI engineering and to actually do research.

> **Status:** Phase 0: setup & baseline. See [`PLAN.md`](PLAN.md) for the roadmap.

## Why

LLM chatbots are tempting research shortcuts, but they **invent citations** and **misattribute claims**.
For students who read English papers in a second language, these errors are even harder to spot.

Research Copilot takes a different approach. **Every claim is traced back to a real passage in a real source,
and then checked.** It is designed as a *coach, not a ghostwriter*. It helps you find, read, and verify sources;
the thinking and writing stay yours.

## What it does (planned)

- **Sharpen research questions.** Turn a vague topic into focused, answerable research questions.
- **Find real papers.** Search OpenAlex, Semantic Scholar, and arXiv in English and Indonesian.
- **Ask your documents.** Ingest PDFs and get answers with citations in the form `[paper, page]` plus the quoted passage.
- **Verify claims.** A checker agent labels each claim as *supported*, *partial*, or *unsupported* by its source.
- **Build an evidence matrix.** Export *claim → source → quote → verdict* to Markdown/CSV.

## Roadmap: AI Engineer levels

| Level | Capability | Key concepts | Status |
|---|---|---|---|
| 1 | LLM chatbot (research coach) | Messages API, prompting, context, tokens & cost | Planned |
| 2 | RAG over your PDFs | Parsing, chunking, embeddings, similarity metrics, vector store, grounding | Planned |
| 3 | Agents with tools | Tool use, agent loop, search agent, checker agent, orchestration, tracing | Planned |

Core mechanisms are implemented **from scratch first**: NumPy vector search before a vector DB, and a hand-written
tool-use loop before any agent framework. The goal is to understand how they work, not just to wire libraries together.

## Research question

Alongside the build, the project evaluates:

1. How often do plain LLM chatbots vs. a retrieval-and-verification copilot produce fabricated or unsupported citations?
2. Does faithfulness differ between Indonesian and English queries?
3. How much time does the copilot save on a real mini-research task?

Results will be published here as the eval runs.

## Architecture (target)

```
            ┌──────────────┐
  topic ──▶ │ Orchestrator │──▶ evidence matrix (claim | source | quote | verdict)
            └──────┬───────┘
       ┌───────────┼────────────┐
       ▼           ▼            ▼
 Search agent   Retriever    Checker agent
 (OpenAlex,     (chunks +    (does the source
  S2, arXiv)    embeddings)   support the claim?)
       │           ▲
       └─▶ ingest ─┘  (PDF → chunks → vectors)
```

## Tech stack

Python 3.11+ · uv · Anthropic Claude API · sentence-transformers (multilingual) · NumPy → ChromaDB/sqlite-vec ·
PyMuPDF · OpenAlex / Semantic Scholar / arXiv APIs · Typer · pytest · ruff

## Project docs

- [`PLAN.md`](PLAN.md): problem, objectives, roadmap, research track
- [`CLAUDE.md`](CLAUDE.md): how AI coding assistants should work in this repo
- `docs/learning-log.md`: what I learned at each milestone
- `docs/decisions.md`: architecture decisions and why

## Author

**Sayyid Thariq**, Information Systems, Universitas Indonesia.
Building toward AI engineering and an MSc in Artificial Intelligence.
