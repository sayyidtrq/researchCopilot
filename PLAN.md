# PLAN.md — Research Copilot

Last updated: 2026-10-10 · Owner: Sayyid

Related docs: [VALUES.md](VALUES.md) (why) · [SYSTEM.md](SYSTEM.md) (FR / NFR / use cases) ·
[DESIGN.md](DESIGN.md) (ERD, use case diagram). Requirement IDs (`UC-xx`, `FR-x.y`) below refer to SYSTEM.md.

## 1. Problem & value

**Problem.** Doing real research (finding papers, reading, extracting evidence, synthesising) is slow and
unstructured for students, so most of us rarely do it. LLM chatbots look like a shortcut, but they
**fabricate citations or misattribute claims**, and readers working in a second language (English papers,
Indonesian writer) are less likely to catch it.

**Who it's for.** First user: me (thesis from Feb 2027 + independent research). Later: other Indonesian students.

**Value.**

| Dimension | What it means | How we measure it |
|---|---|---|
| Time | Topic → evidence-backed outline faster | Minutes per mini-research, manual vs copilot |
| Quality | Real, relevant, verified sources | % citations that exist; % claims supported by cited passage |
| Skill | I get better at research, not dependent | Coach-style features; my own synthesis quality over time |
| Career | AI Engineer portfolio + MSc AI application | Public repo, write-up, demo, eval results |

## 2. Objectives (SMART)

- **O1.** By **31 Jan 2027**, ship an MVP covering AI Engineer Levels 1–3, used to complete **2 real mini-research projects**.
- **O2.** Reduce time from topic to evidence-backed outline by **≥ 40%** vs my manual baseline.
- **O3.** **0 fabricated citations** and **≥ 90% supported claims** on the eval set, versus a plain-LLM baseline.
- **O4.** Publish a technical write-up + learning log by **Feb 2027** (portfolio / MSc application material).

## 3. Scope

The product is an 8-stage research pipeline (see SYSTEM.md §2), each stage ending in a human approval gate:

| Stage | | Release |
|---|---|---|
| S1 Explore topic · S2 Literature · S3 Gap analysis · S4 RQ + method | curiosity → verified gaps → RQ | **MVP** (Levels 1–3, by 31 Jan 2027) |
| S5 Proposal skeleton · S6 Data collection | proposal → dataset | **v2** (thesis period, Feb 2027+) |
| S7 Analysis · S8 Writing support | results → chapter outlines + critique | **v3** (later) |

**In (MVP):** project workspace; research-coach chat; topic candidates; paper search via
OpenAlex / Semantic Scholar / arXiv with explainable ranking; PDF ingest + cited Q&A; concept matrix + gap
analysis; RQ + method suggestions; claim verification; EN + ID; CLI, then simple UI.

**Out (for now):** writing submittable thesis chapters, multi-user/auth, deployment at scale, fine-tuning,
paywalled sources. S5–S8 wait until after the MVP.

## 4. Roadmap

Build incrementally. Each level must be *working and used by me* before starting the next.

### Phase 0 — Setup & baseline (13–19 Oct 2026)

- [ ] Init repo: `uv init`, `ruff`, `pytest`, `.env.example`, `.gitignore`, push to GitHub (scaffold done; push pending)
- [x] Create `docs/learning-log.md` and `docs/decisions.md`
- [x] Draft product docs: `VALUES.md`, `SYSTEM.md` (FR / NFR / use cases)
- [x] Draft `DESIGN.md`: ERD + use case diagram
- [ ] Decide open decisions D1–D5 (SYSTEM.md §8) → `docs/decisions.md`
  - [ ] D1 prose drafting policy (check FASILKOM AI-use policy first)
  - [ ] D2 default LLM (local open model vs API vs Claude Code / Codex CLI)
  - [ ] D3 local DB (SQLite + sqlite-vec vs DuckDB)
- [ ] Pick 3 eval cases (one per research type):
  - [ ] Academic: AI in supply chain management / knowledge management automation
  - [ ] Company case: how Shopee (Sea Ltd) uses AI end-to-end
  - [ ] Policy: US leadership in AI technology
- [ ] **Manual baseline:** do one mini-research by hand, log time spent + pain points (= requirements)
- [ ] Write 20–30 eval questions (EN + ID) with expected source papers → `evals/questions.jsonl`

### Level 1 — LLM chatbot (20 Oct – 2 Nov)

Concepts: Messages API, system prompts, roles, context window, temperature, streaming, tokens & cost.

- [ ] `llm/`: provider interface + first adapter (per D2), retries, token + cost + latency logging (FR-9.3, FR-9.4)
- [ ] Prompts as versioned files; prompt version stored with outputs (FR-9.5)
- [ ] Project workspace: SQLite `project`, `artifact`, `llm_call` tables (FR-0.1, FR-0.2; DESIGN.md §1.3)
- [ ] CLI chat loop with conversation memory (and truncation when context grows)
- [ ] Topic candidates from keywords, with pros/cons + feasibility, save shortlist (UC-02, UC-03; LLM-only for now)
- [ ] Research-coach system prompt: sharpens a topic into SMART research questions, critiques them (UC-07)
- [ ] Structured output: RQ → JSON (RQ, scope, keywords EN/ID, 2–3 method options with rationale) (UC-08)
- [ ] **Baseline experiment:** ask plain LLM for 20 references on each eval case, check how many are real → first data point for the research track
- **Done when:** I use it to sharpen the RQs of all 3 eval cases, and the baseline fabrication rate is recorded.

### Level 2 — RAG: storage & retrieval (3 Nov – 30 Nov)

Concepts: document parsing, chunking strategies, embeddings, similarity metrics (cosine / dot / L2), top-k, metadata, reranking, grounding.

- [ ] Ingest: PDF → text with page numbers (`pymupdf`), metadata (title, authors, year, DOI)
- [ ] Chunking: implement & compare fixed-size, fixed + overlap, and section/paragraph-aware
- [ ] Embeddings: multilingual local model; cache vectors
- [ ] Vector store **from scratch** in NumPy; implement cosine, dot, L2 and compare
- [ ] Swap to ChromaDB or sqlite-vec once the from-scratch version is understood
- [ ] `ask` command: retrieve → answer with inline citations `[paper, p.X]` + quoted passage
- [ ] Concept matrix: dimensions → per-paper cells with quote + page → gaps by type (UC-06, FR-3.x)
- [ ] Knowledge base: ingest UI methodology course material as a separate collection (UC-15, should-have)
- [ ] Retrieval eval: hit@k / MRR on eval questions; pick chunking + metric based on numbers, record in `decisions.md`
- [ ] Test cross-lingual: Indonesian question → English papers
- **Done when:** I can drop 20+ PDFs on a topic and get cited answers I trust, with retrieval metrics logged.

### Level 3 — Agents & tools (1 Dec 2026 – 11 Jan 2027)

Concepts: tool use / function calling, the agent loop (think → act → observe), stopping conditions,
multi-agent orchestration, tracing, guardrails.

- [ ] **Hand-written tool-use loop** on the raw API (no framework), with max-steps + tracing to `data/traces/`
- [ ] Tools: `search_papers` (OpenAlex / Semantic Scholar / arXiv), `fetch_pdf`, `ingest`, `retrieve`, `save_note`
- [ ] **Search agent:** topic → query plan (EN + ID keywords) → search → dedupe → rank → ingest top-N (UC-04)
- [ ] Explainable hybrid ranking (similarity + recency + citations) with score breakdown + "seminal" bucket (FR-2.3, FR-2.4)
- [ ] Ground topic candidates with a quick literature scan: paper count, year trend, seed papers (FR-1.3)
- [ ] **Checker agent:** for each claim + citation: does the source exist? does the passage support it? → supported / partial / unsupported
- [ ] **Orchestrator:** RQ → search agent → retrieve → draft evidence table → checker → report
- [ ] Output: evidence matrix (claim | source | quote | verdict) as Markdown/CSV
- [ ] (Optional) Re-implement one flow with an agent framework and compare — write up the difference
- **Done when:** `rc research "<topic>"` produces a verified evidence matrix for each eval case.

### Evaluation & write-up (12 Jan – 31 Jan 2027)

- [ ] Run full eval: plain LLM vs Level 2 vs Level 3, EN vs ID queries
- [ ] Complete 2 mini-research projects with the copilot, log time vs baseline
- [ ] Technical write-up (blog / README section): architecture, results, lessons
- [ ] Short demo video / GIF
- [ ] Decide: extend into undergraduate thesis? (see Research track)

### v2 — Proposal & data (Feb 2027+, not scheduled yet)

- [ ] Proposal skeleton following the UI course template, with evidence slots (UC-09, FR-5.1)
- [ ] Data plan from RQ + method, with ethics notes (FR-6.1)
- [ ] Connector interface + first connector (per D4: Apify / Crawl4AI / open-data API) (UC-10)
- [ ] Raw JSON + provenance → normalized tables → Excel/CSV export (UC-11)
- [ ] BibTeX / RIS export (UC-17)

### v3 — Analysis & writing support (later)

- [ ] Analysis technique advice + local scripts (UC-12)
- [ ] Chapter outline + critique of my own text (UC-13; scope depends on D1)

## 5. Research track (runs alongside the build)

Working title: *Reducing citation hallucination in LLM research assistants for bilingual (Indonesian–English) students.*

- **RQ1.** How often do plain LLM chatbots vs. a retrieval-and-verification copilot produce fabricated or unsupported citations?
- **RQ2.** Does faithfulness differ between Indonesian and English queries?
- **RQ3.** How much time does the copilot save on a mini-research task, and does it preserve the user's own reasoning?

Method sketch: experimental design (MPPI W#11 — Experiment Research). Conditions = plain LLM / RAG / agent+checker;
metrics = citation existence rate, claim support rate, retrieval hit@k, time-on-task. Start with myself (n=1, pilot),
optionally extend to a small user study for the thesis.

## 6. Risks

| Risk | Mitigation |
|---|---|
| Scope creep / trying everything at once | One level at a time; "done when" criteria |
| Letting AI write the code without understanding it | Learning mode in CLAUDE.md; learning log per milestone |
| API cost | Log cost per call; use small models for dev; cache |
| Paywalled PDFs | Use open-access (OpenAlex OA links, arXiv); manual upload otherwise |
| Scope creep from S5–S8 vision | MVP is S1–S4 only; S5–S8 are v2/v3 |
| Scraping breaks ToS / UU PDP | Prefer official APIs; ethics note + consent flag per data source |
| Campus rules on AI use in thesis | Check faculty policy before using it for thesis work; tool is assistant, not author |

## 7. Progress log

| Date | Update |
|---|---|
| 2026-10-10 | Project defined: problem, value, objectives, L1–L3 roadmap, research track. |
| 2026-10-10 | Phase 0 scaffold: uv project, pytest, ruff, `.env.example`, docs. |
| 2026-10-10 | Product defined end-to-end: VALUES.md, SYSTEM.md (8 stages, 17 use cases, FR/NFR), DESIGN.md (ERD, use case diagram). MVP = S1–S4. |
