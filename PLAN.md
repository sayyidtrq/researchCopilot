# PLAN.md — Research Copilot

Last updated: 2026-10-10 · Owner: Sayyid

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

**In:** chat with a research-coach persona; ingest PDFs; retrieval with citations; paper search via
OpenAlex / Semantic Scholar / arXiv; claim verification; EN + ID queries; CLI, then simple UI.

**Out (for now):** writing thesis chapters, multi-user/auth, deployment at scale, fine-tuning, paywalled sources.

## 4. Roadmap

Build incrementally. Each level must be *working and used by me* before starting the next.

### Phase 0 — Setup & baseline (13–19 Oct 2026)

- [ ] Init repo: `uv init`, `ruff`, `pytest`, `.env.example`, `.gitignore`, push to GitHub (scaffold done; push pending)
- [x] Create `docs/learning-log.md` and `docs/decisions.md`
- [ ] Pick 3 eval cases (one per research type):
  - [ ] Academic: AI in supply chain management / knowledge management automation
  - [ ] Company case: how Shopee (Sea Ltd) uses AI end-to-end
  - [ ] Policy: US leadership in AI technology
- [ ] **Manual baseline:** do one mini-research by hand, log time spent + pain points (= requirements)
- [ ] Write 20–30 eval questions (EN + ID) with expected source papers → `evals/questions.jsonl`

### Level 1 — LLM chatbot (20 Oct – 2 Nov)

Concepts: Messages API, system prompts, roles, context window, temperature, streaming, tokens & cost.

- [ ] `llm/client.py`: raw Anthropic API call, retries, token + cost + latency logging
- [ ] CLI chat loop with conversation memory (and truncation when context grows)
- [ ] Research-coach system prompt: sharpens a vague topic into SMART research questions, asks follow-ups
- [ ] Structured output: research question → JSON (RQ, scope, keywords EN/ID, candidate methods)
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
- [ ] Retrieval eval: hit@k / MRR on eval questions; pick chunking + metric based on numbers, record in `decisions.md`
- [ ] Test cross-lingual: Indonesian question → English papers
- **Done when:** I can drop 20+ PDFs on a topic and get cited answers I trust, with retrieval metrics logged.

### Level 3 — Agents & tools (1 Dec 2026 – 11 Jan 2027)

Concepts: tool use / function calling, the agent loop (think → act → observe), stopping conditions,
multi-agent orchestration, tracing, guardrails.

- [ ] **Hand-written tool-use loop** on the raw API (no framework), with max-steps + tracing to `data/traces/`
- [ ] Tools: `search_papers` (OpenAlex / Semantic Scholar / arXiv), `fetch_pdf`, `ingest`, `retrieve`, `save_note`
- [ ] **Search agent:** topic → query plan (EN + ID keywords) → search → dedupe → rank → ingest top-N
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
| Campus rules on AI use in thesis | Check faculty policy before using it for thesis work; tool is assistant, not author |

## 7. Progress log

| Date | Update |
|---|---|
| 2026-10-10 | Project defined: problem, value, objectives, L1–L3 roadmap, research track. |
| 2026-10-10 | Phase 0 scaffold: uv project, pytest, ruff, `.env.example`, docs. |
