# CLAUDE.md

> Claude Code reads this file. The canonical, harness-agnostic instructions live in
> [`AGENTS.md`](AGENTS.md) — read it before working on this repo.
> English version. Русская версия: [CLAUDE.ru.md](CLAUDE.ru.md).

---

This project keeps a **single source of truth** for AI assistants in
[`AGENTS.md`](AGENTS.md). It is intentionally not duplicated here to avoid drift.

**Before doing anything else:**
1. Read [`AGENTS.md`](AGENTS.md) — project overview, conventions, rules for AI assistants.
2. Read the relevant document in [`docs/`](docs/) **before** opening source files:
   - [`docs/01_project_structure.md`](docs/01_project_structure.md) — full directory tree
   - [`docs/02_architecture.md`](docs/02_architecture.md) — architecture & patterns
   - [`docs/03_execution_flow.md`](docs/03_execution_flow.md) — execution flow
   - [`docs/04_code_quality.md`](docs/04_code_quality.md) — code quality audit & known gotchas
   - [`docs/05_optimization_roadmap.md`](docs/05_optimization_roadmap.md) — optimization roadmap
3. Do **not** recursively scan the whole `backend/src/` or `frontend/src/` — the tree is
   already documented in `docs/01`.

Key rules (full list in `AGENTS.md`): make minimal changes, keep backend async, trust
`docs/` over `README.md`/`ARCHITECTURE.md`, do not edit `docs/` unless asked, do not
commit/push without confirmation.
