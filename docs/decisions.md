# Decisions

Notable architecture and tooling decisions, newest last.

## 2026-10-10 — Project scaffold

- **uv + src layout + hatchling** build backend; package `research_copilot` under `src/`.
- **Dev tools only for now:** `pytest`, `ruff`. Runtime deps (`anthropic`, `typer`, `pymupdf`,
  `sentence-transformers`, ...) are added when the milestone that needs them starts, not up front.
- **Ruff rules:** `E, F, I, UP, B`, line length 100.
