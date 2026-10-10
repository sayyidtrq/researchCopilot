# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Research Copilot** is a bilingual (English / Bahasa Indonesia), *verifiable* AI research assistant.
It helps its user find real papers, read them, extract claims, and check that every claim is
actually supported by its cited source.

It has three goals, in this order of priority:

1. **Learning AI engineering from first principles.** The owner (Sayyid, Information Systems student,
   Universitas Indonesia) is using this project to learn how LLM apps, RAG, and agents work *under the hood*.
2. **Being genuinely useful** for his own research: undergraduate thesis (starting Feb 2027) and
   independent research (e.g. AI in supply chain, knowledge management, company case studies).
3. **Portfolio + research output** for an AI Engineer career and an MSc AI application (University of Edinburgh).
   The project doubles as a research experiment on citation faithfulness (see `PLAN.md` → Research track).

## Project docs (read before designing anything)

| File | Answers | Notes |
|---|---|---|
| `VALUES.md` | *Why*: user, jobs-to-be-done, value, principles | Product source of truth |
| `SYSTEM.md` | *What*: 8-stage pipeline, use cases `UC-xx`, requirements `FR-x.y` / `NFR-x`, open decisions `D1–D5` | Reference IDs in commits and `PLAN.md` |
| `DESIGN.md` | *How*: ERD, use case diagram, design decisions | Update when the schema changes |
| `PLAN.md` | *When*: roadmap, checkboxes | MVP = stages S1–S4 |

- These docs are edited in **Obsidian**: keep YAML frontmatter, `[[wikilinks]]`, `> [!callout]` blocks, and
  Mermaid diagrams in `VALUES.md`, `SYSTEM.md`, `DESIGN.md`. `PLAN.md`, `README.md`, and `CLAUDE.md` use plain
  Markdown links (they must render on GitHub).
- `VALUES.md`, `SYSTEM.md`, `DESIGN.md` are currently gitignored (private drafts); `.obsidian/` is gitignored.

## How to work with me (learning mode — important)

- **Explain before you build.** For any core concept (LLM call, chunking, embeddings, similarity metric,
  vector search, tool-use loop, agent orchestration), briefly explain the idea and the trade-offs first,
  then implement. Keep explanations short and concrete.
- **From scratch first, libraries second.** Implement the core mechanism by hand before reaching for a
  library. Examples: cosine similarity in NumPy before a vector DB; a hand-written tool-use loop before
  any agent framework. **Do not use LangChain, LlamaIndex, CrewAI, or similar frameworks** for Level 1–3 core logic
  unless I explicitly ask. A later "compare with framework X" exercise is fine.
- **Small steps.** One milestone (see `PLAN.md`) at a time. Prefer small, reviewable changes.
  Stop and summarise what was done and what I should understand before moving on.
- **Ask before adding dependencies** or changing the architecture. Record notable decisions in `docs/decisions.md`.
- **Update the learning log.** After finishing a milestone, append a short entry to `docs/learning-log.md`:
  what was built, the key concept, one thing that surprised us, open questions.
- **Tick checkboxes in `PLAN.md`** when a task is done.
- I may chat in Indonesian; **code, comments, commit messages, and docs are in English.**

## Product principle: coach, not ghostwriter

The copilot helps me *do* research, it does not do research *for* me.
It may: sharpen research questions, find and rank sources, extract and quote passages, verify claims,
critique my synthesis. It should not: write my thesis chapters or invent arguments.
**Every factual claim it outputs must link to a real source passage** (paper + page/section + quote).
Never fabricate a citation; if nothing supports a claim, say so.

- Every pipeline stage ends in a **human approval gate**; never auto-advance a project to the next stage.
- Proposal and chapter support (stages S5, S8) produce **skeletons + critique**, not prose, until decision
  D1 in `SYSTEM.md` is made and recorded in `docs/decisions.md`.
- Data collection must respect ToS / robots.txt / rate limits and UU PDP (personal data needs a consent note).

## Tech stack (initial — may change, record changes in docs/decisions.md)

- Python 3.11+, managed with `uv`
- LLM: behind one provider interface (`llm/`). Dev default: a free open model run locally. Optional adapters:
  Claude Code CLI (`claude -p`) and Codex CLI (`codex exec`) via the user's subscriptions, and HTTP APIs.
  Use official CLIs only; never extract session tokens. Final default = decision D2.
- Storage: SQLite single file `data/copilot.db` (schema in `DESIGN.md`); raw collected data as immutable files
- Embeddings: local multilingual model via `sentence-transformers` (e.g. `intfloat/multilingual-e5-small`) — supports Indonesian
- Vector store: NumPy (from scratch) first, then ChromaDB or sqlite-vec
- PDF parsing: `pymupdf`
- Paper search APIs: OpenAlex, Semantic Scholar, arXiv
- CLI: `typer`; a simple UI (Streamlit) only after Level 2 works
- Tests: `pytest`; lint/format: `ruff`

## Repository layout (target)

```
src/research_copilot/
  llm/          # provider interface + adapters (local, Claude Code CLI, Codex CLI, API), call logging
  store/        # SQLite schema + data access (projects, artifacts, papers, ...)
  chat/         # Level 1: chatbot, conversation memory, prompts, topic + RQ coach
  rag/          # Level 2: ingest, chunking, embeddings, vector store, retrieval, concept matrix
  agents/       # Level 3: tool-use loop, tools/, search + checker agents
  connectors/   # v2: data collection connectors (Apify, open-data APIs, scrapers)
  cli.py
evals/          # eval cases, question sets, scripts, results
data/           # PDFs, indexes, caches (gitignored)
docs/           # learning-log.md, decisions.md, research notes
tests/
```

## Commands (once scaffolded)

```bash
uv sync                      # install deps
uv run rc chat               # Level 1 chat
uv run rc ingest <pdf|dir>   # Level 2 ingest documents
uv run rc ask "<question>"   # Level 2 RAG answer with citations
uv run rc research "<topic>" # Level 3 agentic research
uv run pytest                # tests
uv run ruff check . && uv run ruff format .
uv run python evals/run_eval.py
```

## Rules

- Secrets live in `.env` (see `.env.example`). Never commit keys, PDFs, or vector indexes.
- Log every LLM call (model, tokens in/out, latency, cost estimate) — it's needed for evaluation.
- Keep prompts in versioned files or constants, not scattered inline strings.
- No `Co-Authored-By: Claude` (or any AI attribution) lines in commit messages or PR descriptions.
- Respect API rate limits and terms of OpenAlex / Semantic Scholar / arXiv; cache responses in `data/cache/`.
- Stay inside the MVP (stages S1–S4) unless I explicitly start v2 work.
